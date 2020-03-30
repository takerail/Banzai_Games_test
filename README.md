Тестовое задание для BANZAI.GAMES

Задача 1:
Изучите перк TOUGH_BODY и создайте перк, который с вероятностью 
1/4 в 3 раза уменьшает урон, полученный хозяином перка от критического удара в голову.

Решение: 
Задача решается путем копирования перка TOUGH_BODY со следующими изменениями: 
- переименование перка и его единственного триггера в TOUGH_HEAD
- константа defenseDamageFactor изменена на -15850 (log от 1/3 по основанию 2)
- константа defense изменена на "HeadDefense" (удар в голову)
- константа chance изменена на 0.25 (шанс 1/4)
- в условии срабатывания триггера TOUGH_HEAD удалено свойство event.Block и добавлено свойство event.Critical (т.к заблокированный удар
не может быть критическим, следовательно свойство блока излишне)

Задача 2:
Изучите перк FITNESS и измените его так, чтобы он, не меняя вероятности срабатывания
при ударе без оружия, с вероятностью 0.02 в 2 раза увеличивал урон по противнику любым ударом.

Решение:
Задача была решена путем добавления нового триггера для удара с оружием:
- для удобства константа attackBoostChance переименована в unarmedAttackBoostChance
- триггер FITNESS_ATTACK переименован в FITNESS_UNARMED_ATTACK
- добавлена констата weaponAttackBoostChance = 0.02
- добавлен триггер FITNESS_WEAPON_ATTACK, идентичный триггеру FITNESS_UNARMED_ATTACK, за исключением шанса срабатывания (weaponAttackBoostChance) и свойства event.Animation=="Weapon"
Таким образом, для данного перка, при срабатывание события "PreHit" проверяются условия для двух триггеров:
1) Если удар совершен без оружия - с шансом 0.3 срабатывает триггер FITNESS_UNARMED_ATTACK
2) Если удар совершен с оружием - с шансом 0.02 срабатывает триггер FITNESS_WEAPON_ATTACK

Возможно задачу следовало решить путем изменения условия триггера FITNESS_ATTACK на следующее:
Condition
    {
      return OR (AND ( event.Target=="Enemy", event.Animation=="Unarmed", random() < unarmedAttackBoostChance )),
      (AND ( event.Target=="Enemy", event.Animation=="Weapon", random() < weaponAttackBoostChance ))
    }
Но на мой взгляд предложенное мной решение нагляднее, хоть и более громоздко
