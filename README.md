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

Примечание:
Возможно задачу следовало решить путем изменения условия триггера FITNESS_ATTACK на следующее:

Condition
    {
      return OR (AND ( event.Target=="Enemy", event.Animation=="Unarmed", random() < unarmedAttackBoostChance )),
      (AND ( event.Target=="Enemy", event.Animation=="Weapon", random() < weaponAttackBoostChance ))
    }
    
Но на мой взгляд предложенное мной решение нагляднее, хоть и более громоздко

Задача 3:
Изучите HELM_BREAKER. Измените его так, чтобы он не мог сработать в первые
10 секунд раунда, а действие усиления в первую секунду после приобретения было
удвоено.

Решение:
Задача была решена следующим путем:
- добавлен триггер HELM_BREAKER_INITIAL_RESTRICTION, всегда вызываемый при срабатывания события "FightStart", этот триггер создает иконку баффа "Head_Breaker_restriction" над врагом, которая запрешает срабатывание триггера перка, ответсвенного за накладывание дебаффа на противника
- добавлен триггер HELM_BREAKER_INITIAL_BOOST, который срабатывает с шансом chance = 0.2 если на противнике не присуствуют эффекты данного перка ("Head_Breaker_restriction", "Initial_Boost_Icon", "Icon"), удар приходится в голову и не заблокирован. Этот триггер накладывает на врага эффект "Initial_Boost_Icon", который длится initialBoostFrames = 60 (1 секунду)
- добавлен триггер HELM_BREAKER_INITIAL_BOOST_END, который срабатывает при любом незаблокированном ударе в голову противника, если при этом на него был наложен эффект "Initial_Boost_Icon" и при срабатывании увеличивает урон от удара на attackDamageFactor*2
- изменена логика поведения триггера HELM_BREAKER_BOOST, теперь он срабатывает при событии "ModExpires", если event.Name="Initial_Boost_Icon" (т.е при истечении 1 секунды после приобретения эффекта) и накладывает старый бафф "Icon", который длится iconFrames - initialBoostFrames (т.е 300 фреймов - 60 фреймов = 240 фреймов, или 4 секунды)
- триггер HELM_BREAKER_BOOST_END оставлен без изменений: он срабатывает при любом незаблокированном ударе в голову противника, если при этом на него был наложен эффект "Icon" и при срабатывании увеличивает урон от удара на attackDamageFactor

Примечание: 
В этой задаче наиболее заметна возможная проблема с производительностью - большинство условий триггеров возврашают результат работы логического И, а в большинстве случаев, как мне известно, этот оператор работает последовательно, и возвращает false при первом появлении аргумента равного false. Например условие

AND ( event.Player="Enemy", event.Defense=defense, not event.Block, ModExists ( Name = "Icon" ) )

при незаблокированном ударе выглядит следующим образом

true && true && true && false = false

При большом количестве условий это скорее всего вызовет проблемы с производительностью, для исправления которых мы можем изменить условие на следующее

AND ( ModExists ( Name = "Icon" ), event.Player="Enemy", event.Defense=defense, not event.Block )

при незаблокированном ударе оно вернет

fasle && ... = false

т.е не будет проверять остальные условия, если первое заведомо ложно.
Значит, если нам известно, что некоторые условия нет смысла проверять ( в данном случае нет смысла проверять куда пришелся удар, был ли удар сделан по противнику и т.д. так как нам известно, что эффект перка отсуствует на игровой арене), то желательно изменить порядок условий с учетом их значимости. Однако, поскольку мне достоверно неизвестно прав ли я, а во всем файле условия представлены в таком виде, далее я продолжу использовать именно такие условия.


Задача 4:
Изучите перк RAGE. Корректен ли он? Если нет, то укажите, в чем заключается ошибка и постарайтесь исправить её.

Решение:
судя по коду этот перк должен функционировать следующим образом:
При любом ударе, перед подсчетом урона, с шансом 0.3 на владельца перка накладывается новый экземпляр баффа, незначительно увеличивающего урон от ударов. Этих баффов может накладываться неограничено много (возможно это плохо с точки зрения баланса), пока игроку везет. Как только шанс наложения баффа не срабатывает (random()>=0.3), с персонажа снимаются все "стаки" баффа и его иконка.

В данной реализации присуствует некорректное поведение: сперва проверяется random()<0.3 и при срабатывание создается событие "success",
затем сразу же проверяется уже другое значение random()>=0.3 (значение функции random() отличается от того значения, что эта функция вернула в первом триггере), и если эта функция вернула true, то создается событие "fail".
Самым простым способом решения этой проблемы по моему мнению являются следующие изменения:
- В блок action триггера RAGE_CHECK_SUCCESS добавлена функция ModFlag (Name = "Rage_success", Frames=1), т.е. при успешной проверке condition на персонажа вешается эффект "Rage_success" на один кадр
- Триггер RAGE_CHECK_FAIL удаляется
- на место предыдущего триггера переносится RAGE_FAIL, event этого триггера заменяется на return "PreHit", чтобы он срабатывал каждый удар, condition меняется на return not ModExists(Name = "Rage_success"), чтобы триггер срабатывал только если в этот кадр не сработал триггер RAGE_CHECK_SUCCESS, action остается без изменений (т.е удаляются экземпляры баффа и его иконка)
