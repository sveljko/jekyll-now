---
layout: post
title: Потпуна покривеност, непотпуно тестирање
lang: sr
---

Покривеност кода је корисна метрика тестирања, али, није _циљ_. Код
може да има 100% покривености, па да ипак има буба и може да има 0%
покривеност - а нема ниједну једину бубу.

## 100% покривен код са бубом

У модулу који је имао 100% покривености, како редова кода, тако
и грана, постојала је прилично озбиљна буба.

Модул се бавио листама временских контрола (ВК), у виду двоструког
повезаних листи. Сваки члан носи податак "колико тикова треба да
прође од истека претходног члана док ми не истекнемо". Ово чини
обраду листа ВК прилично ефикасном. "Тик" је јединица мере времена
у датој примени (ms је честа).

Буба је проста - ако укидамо ВК, дакле изланчавамо је из лист,
а она је _прва_ у листи (следећа да истекне), то није обрађивано
исправно. Време да та ВК истекне _није_ додавано на време за
следећу ВК.

Ради илустрације, претпоставимо да има две ВК, обе покренуте да трају
100 тикова, али, са једним тиком размака. Дакле, по покретању друге
ВК, листа је изгледала овако:

    T1(99) -> T2(1)

Након што је прошло неких 40 тикова, листа би била:

    T1(59) -> T2(1)

Онда је Т1 укинута, па би листа _требала_ да буде:

    T2(60)
	
Али, због бубе, листа је у ствари била:

    T2(1)
	
Односно, Т2 ће да истекне након 41 тика, уместо 100 тикова.

Постојала је грана која изричито обрађује овај случај. Али,
код који додаје "време до истека" просто није постојао. Ради
се о Ц-у, а ред кода који је недостајао је био:

    list->timeout_left_ms += to_remove->timeout_left_ms;

## Зашто ("потпуни") тестови ово нису ухватили?

Тестови _јесу_ тестирали избацивање првог члана листе. Али, пошто је
`timeout_left_ms` унутрашња ствар модула, тестови то нису проверавали.
Такође, нису проверавали ни "шта ко прође неко време након избацивања".
У нашем случају, ако после избацивања прође 40 тикова, Т2 _не_ треба
да истекне (али у ствари јесте истицао).

Очигледно, овај прост сценарио није уочен при тестирању, него је свака
могућност модула независно тестирана. Тестови за укидање су тестирали
само изланчавање. Тестови за "пролазак времена" су тестирали само
време. Бубе се врло често крију у сценаријима који обухватају више
могућности.


## Наравоученије

За високо квалитетне тестове, треба увидети сценарије упторебе и 
тестирати их (све, по могућству). Управо _не_ треба размишљати
о покривености кода. Штавише, висока покривеност _не_ треба да буде
циљ. Не желимо да "покријемо" код, већ желимо да га _тестирамо_.
Мера покривености је само врста "назнаке" - ако је покривеност
ниска, вероватно тестови нису баш најбољи.