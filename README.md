# Uppaal models

We considered 27 meta-configurations of SAI models.
There are three catalogs with the models:
* *concrete* - original models,
* *abstract-a* - abstract models (after applying A1),
* *abstract-b* - abstract models (after applying A2).


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
