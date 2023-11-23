# Uppaal models

We considered 27 meta-configurations of SAI models.
There are three catalogs with the models:
* *concrete* - original models,
* *abstract-a* - abstract models (after applying A1),
* *abstract-b* - abstract models (after applying A2).

## Abstraction parameters

In order to generate the overapproximating may-abstractions we used the following script iteratively calling to `app.js` from EasyAbstraction4Uppaal tool:
```sh
# set the path prefix
MODELS_PREFIX='./concrete'
# iterate over concrete models
for f in $(ls "$MODELS_PREFIX" | grep -P '.xml$'); do
    # remove invariants
    "./$MODELS_PREFIX/../rem-inv.sh" "$MODELS_PREFIX/$f"
    #--------------------------------------------------
    # update the abstraction config
    ./app.js config input="$MODELS_PREFIX/$f" output="$MODELS_PREFIX/../abstract-a/${f%.xml}-abstract-a.xml" dmap="$MODELS_PREFIX/../misc/d-a-${f%.xml}.txt"
    ./app.js approx 1 targetVars=completion,mstatus initVal=0,0 targetTemplate=AI 
    ./app.js abstract targetVars=completion,mstatus initVal=0,0 targetTemplate=AI 
    # --------------------------------------------------
    # update the abstraction config
    ./app.js config input="$MODELS_PREFIX/$f" output="$MODELS_PREFIX/../abstract-b/${f%.xml}-abstract-b.xml" dmap="$MODELS_PREFIX/../misc/d-b-${f%.xml}.txt"
    ./app.js approx 1 targetVars=data,completion,mstatus,info initVal=0,0,0,0 targetTemplate=AI 
    ./app.js abstract targetVars=data,completion,mstatus,info initVal=0,0,0,0 targetTemplate=AI 
    #--------------------------------------------------
    # restore invariants for concrete
    "./$MODELS_PREFIX/../res-inv.sh" "$MODELS_PREFIX/$f"
    # restore invariants for abstract
    "./$MODELS_PREFIX/../res-inv.sh" "$MODELS_PREFIX/../abstract-a/${f%.xml}-abstract-a.xml"
    "./$MODELS_PREFIX/../res-inv.sh" "$MODELS_PREFIX/../abstract-b/${f%.xml}-abstract-b.xml"
done
```

The content of utility scripts that remore and restore auxiliary invariants over all locations is as follows:
```sh
# rem-inv.sh
sed -i '/<label kind="invariant"./d' $1
sed -i '/clock x/d' $1
# res-inv.sh
sed -i '/<\/location>/i\<label kind="invariant">x&lt;=0</label>/' $1
sed -i '/const int NA/i\clock x;' $1
```


# Queries

The Uppaal queries used in verification are as follows:
* deadlock-freeness  
`true --> exists(i:int[1,NA])(AI(i).wait)`
* eventually-win  
`A<>((sum(i:int[1,NA])AI(i).mqual)>0)`
* flawless-win (meta-configurations with impersonator-attacker)  
`forall(i:int[1,NA])(impersonated!=i imply AI(i).mstatus==2)-->((forall(i:int[1,NA])(!AI(i).wait)) imply (!(forall(i:int[1,NA])(impersonated!=i imply AI(i).mstatus==2)) || ((sum(i:int[1,NA])AI(i).mqual)>0)))`
* flawless-win (meta-configurations without impersonator-attacker)  
`forall(i:int[1,NA])(AI(i).mstatus==2)-->((forall(i:int[1,NA])(!AI(i).wait)) imply (!(forall(i:int[1,NA])(AI(i).mstatus==2)) || ((sum(i:int[1,NA])AI(i).mqual)>0)))`
