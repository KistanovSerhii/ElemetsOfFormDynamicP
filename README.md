# ElemetsOfFormDynamicP

1С Elemets of Form with dynamic properties / Динамическая установка свойств элементам формы
Обработка позволяет программно задать значение любому свойству элемента формы.
(Ширина, Высота, Доступность, ТолькоПросмотр, Видимость, Шрифт и т.д.)

# Почему не типовое решение или БСП:
Типовое решение позволяет управлять элементом как ВидимостьюПоРолям или БСП которая дает возможность "ЗаблокироватьРеквизиты",
но дело в том что они не дают возможность управлять любым свойством конкретного элемента формы.

# Краткое описание вызова:

	1. форма - СсылкаНаБлокируемуюформу,
	2. свойстваЭлементов - Массив ИЗ Структура (имя элемента, свойство и его значение которые будут установлены)
	3.1 свойстваЭлементовОбщее - Структура описывает одно свойство и знч которое будет задано ВСЕМ элементам формы!

Обработка.УстановитьСвойствоЭлементаФормы(форма, свойстваЭлементов, свойстваЭлементовОбщее, видыИТипы);

	3.2 свойстваЭлементовОбщее - например всем элементам устанавливаем "ТолькоПросмотр" Истина, а
	свойстваЭлементов - добавляем элемент "Наименование" и "Родитель" которым устанавливаем свойство "ТолькоПросмотр" в Ложь
	и таким образом мы запрещаем редактировать ВСЕ элементы кроме "Наименование" и "Родитель"!

 	4. видыИТипы - это Соответствие (не обязательный параметр), которое перечислят какие типы элементов формы обрабатывать.
  	Если не передавать тогда будет как видыИТипы = внешняяОбработкаОбъект.НаборВидовИТиповЭлементовФормы();
   	*Например: ВидПоляФормы.ПолеВвода, ВидКнопкиФормы.КнопкаКоманднойПанели, ВидПоляФормы.ПолеФлажка и т.д.

# (Пример) Задача: 

1. Все реквизиты формы справочника Сотрудники НЕ должны быть доступны для редактирования
и только те пользователи которые входят в группу доступа "ОграниченныйДоступСправочникСотрудники
могут менять значение реквизита Организация и команда "Записать и закрыть"!

2. Пользователи входящие в группу доступа "ОграниченныйДоступСправочникСотрудникиРасширенный" должны
могут менять значение реквизита Организация и команда "Записать и закрыть", а также остальные реквизиты должны быть
доступны, но только на просмотр (имеется ввиду: ТолькоПросмотр = Истина, Доступность = Истина)!

3. Пользователи входящие в группу доступа "Администраторы" должны не должны иметь ограничений!

# Внедрение:

ВАЖНО: Рекомендую использовать с проверкой вхождения в ГруппыДоступа, см. Область ПроверитьВхождениеВПрофиль_СвойствЭлементовФормы.
Добавьте код из раздела (Дополнительно) который проверит вхождение в ГруппыДоступа!


1. Открывает Модуль формы в основной конфигурации (если редактирование разрешено) или в расширения (если редактирование основной конфигурации запрещено)
2. Переходим в событие "ПриСозданииНаСервере"
3. Копируем (Ctrl + C -> Ctrl + V) код:

		#Область НовыеЗнчДляСвойствЭлементовФормы
		#Область ПолучениеВнешнейОбработки_СвойствЭлементовФормы

		УстановитьПривилегированныйРежим(Истина);
		внешняяОбработкаИмя    = "ДинамическоеСвойствоЭлементаФормы";
		внешняяОбработкаСсылка = Справочники.ДополнительныеОтчетыИОбработки.НайтиПоРеквизиту("ИмяОбъекта", внешняяОбработкаИмя);
		внешняяОбработкаОбъект = ДополнительныеОтчетыИОбработки.ОбъектВнешнейОбработки(внешняяОбработкаСсылка);
		УстановитьПривилегированныйРежим(Ложь);

		#КонецОбласти

		#Область ПроверитьВхождениеВПрофиль_СвойствЭлементовФормы

		мГруппыДоступа = Новый Массив;
		мГруппыДоступа.Добавить("ОграниченныйДоступСправочникСотрудники");
		доступРазрешен = внешняяОбработкаОбъект.ТекущийПользовательДобавленВГруппыДоступа(мГруппыДоступа);

		#КонецОбласти

		// Описываете каким элементам формы какое значение хотите установить +++
		свойстваЭлементовОбщее = Новый Структура("Свойство,Значение", "Доступность", Ложь);

		свойстваЭлементов = Новый Массив; 
		свойстваЭлементов.Добавить( Новый Структура("Имя,Свойство,Значение", "ФормаЗаписатьИЗакрыть", "Доступность", доступРазрешен) );
		свойстваЭлементов.Добавить( Новый Структура("Имя,Свойство,Значение", "ГоловнаяОрганизация", "Доступность", доступРазрешен) );
		свойстваЭлементов.Добавить( Новый Структура("Имя,Свойство,Значение", "ФИО", "ТолькоПросмотр", доступРазрешен) );
		// Описываете каким элементам формы какое значение хотите установить +++

		#Область Выполнение_СвойствЭлементовФормы
		внешняяОбработкаОбъект.УстановитьСвойствоЭлементаФормы(ЭтаФорма, свойстваЭлементов, свойстваЭлементовОбщее);
		#КонецОбласти

		#КонецОбласти


# (Дополнительно) код проверки вхождения в Группу доступа:

Как правило управлять значением свойства элемента надо на основании вхождения пользователя в Грпуппу доступа

	#Область ТекущийПользовательДобавленВГруппыДоступа

	// мГруппыДоступа - Массив ИЗ Строка - Наименование ГруппыДоступа
	//
	// Возврат Булево
	Функция ТекущийПользовательДобавленВГруппыДоступа(мГруппыДоступа) Экспорт

	УстановитьПривилегированныйРежим(Истина);
	Запрос = Новый Запрос;
	Запрос.Текст = 
	
	"Выбрать 
	|	Пользователь 
	|ИЗ 
	|	Справочник.ГруппыДоступа.пользователи
	|Где
	|	Пользователь = &ТекущийПользователь
	|	И Ссылка.Профиль.Наименование В (&ИмяПрофиля)";
	
	Запрос.УстановитьПараметр("ТекущийПользователь",	ПараметрыСеанса.ТекущийПользователь);
	Запрос.УстановитьПараметр("ИмяПрофиля", 		мГруппыДоступа);
	
	Выборка = Запрос.Выполнить().Выбрать();
	
	Возврат (Выборка.Следующий() И Выборка.Пользователь <> "");
		
	КонецФункции

	#КонецОбласти

Соответственно создайте ГруппыДоступа с наименованием например "ОграниченныйДоступСправочникСотрудники" и
добавьте нужных пользователей (ГруппыДоступа может быть без ролей так как мы используем ГруппыДоступа только для проверки наличия в них пользователя)
