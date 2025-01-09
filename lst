import sqlite3
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes, CallbackQueryHandler, ConversationHandler
import logging
from datetime import datetime



logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)

# Определяем состояния
SELECT_PROFESSION, GET_NAME, TASKS = range(3)

CHECKLIST_TASKS = [
    {
        'id': 1,
        'title': "Отключить вывеску",
        'text': "Доброе утро, {name}! Добро пожаловать на работу! Пожалуйста, выключите вывеску и сделайте фотографию выключателя.",
        'type': 'photo'
    },
    {
        'id': 2,
        'title': "Включить кондиционер",
        'text': "Включите кондиционер в соответствующий времени года режим, приложите фото.",
        'type': 'photo'
    },
    {
        'id': 3,
        'title': "Включить вытяжку",
        'text': "Включите вытяжку, сделайте фото. Для вашего удобства по дороге включите компьютер.",
        'type': 'photo'
    },
    {
        'id': 4,
        'title': "Включить музыку",
        'text': "Пожалуйста, включите музыку, сделайте фото микшера.",
        'type': 'photo'
    },
    {
        'id': 5,
        'title': "Надеть рабочую форму",
        'text': "Наденьте рабочую форму, сделайте фото.",
        'type': 'photo'
    },
    {
        'id': 6,
        'title': "Подготовить вчерашний хлеб",
        'text': "Возьмите вчерашний хлеб и выставите его на витрину. Приложите фото витрины.",
        'type': 'photo'
    },
    {
        'id': 7,
        'title': "Открыть смену в IIKO",
        'text': "Откройте смену в IIKO, сделайте внесение, приложите фото.",
        'type': 'photo'
    },
    {
        'id': 8,
        'title': "Проверить чистоту зала",
        'text': "Убедитесь, что зал чист, столы протерты, подстолья чистые, стулья стоят на своих местах. Проверьте диван и 4 мягких стула на наличие волос и/или крошек! Сделайте фото зала.",
        'type': 'photo'
    },
    {
        'id': 9,
        'title': "Проверить сервировочный стол",
        'text': "Убедитесь, что сервировочный стол чист, приборов хватает, стаканчики с салфетками заполнены. Сделайте фото.",
        'type': 'photo'
    },
    {
        'id': 10,
        'title': "Температура бОльшей витрины",
        'text': "Сколько градусов температура первой (бОльшей) витрины? Напишите цифрами.",
        'type': 'text'
    },
    {
        'id': 11,
        'title': "Температура малой витрины",
        'text': "Сколько градусов температура второй (малой) витрины? Напишите цифрами.",
        'type': 'text'
    },
    {
        'id': 12,
        'title': "Протереть бОльшую витрину",
        'text': "Протрите первую (бОльшую) витрину, убедитесь в наличии всех позиций, также проверьте товарный вид. Сделайте фото.",
        'type': 'photo'
    },
    {
        'id': 13,
        'title': "Протереть малую витрину",
        'text': "Протрите вторую (малую) витрину, убедитесь в наличии всех позиций, проверьте товарный вид. Сделайте фото.",
        'type': 'photo'
    },
    {
        'id': 14,
        'title': "Проверить бар (до бойлера)",
        'text': "Убедитесь в чистоте бара. Сделайте фото от весов до бойлера.",
        'type': 'photo'
    },
    {
        'id': 15,
        'title': "Проверить бар (бойлер–раковина)",
        'text': "Убедитесь в чистоте бара 2. Сделайте фото от бойлера до раковины.",
        'type': 'photo'
    },
    {
        'id': 16,
        'title': "Проверить зону бариста",
        'text': "Убедитесь в чистоте зоны бариста. Приложите фото.",
        'type': 'photo'
    },
    {
        'id': 17,
        'title': "Разложить новый хлеб",
        'text': "По прибытию свежего хлеба разложите новый хлеб на витрину, сделайте фото.",
        'type': 'photo'
    },
    {
        'id': 18,
        'title': "Позиции на стопе (кухня)",
        'text': "Напишите, какие позиции на стопе по кухне.",
        'type': 'text'
    },
    {
        'id': 19,
        'title': "Позиции на стопе (бар)",
        'text': "Напишите, какие позиции на стопе по бару.",
        'type': 'text'
    },
    {
        'id': 20,
        'title': "Завершение чек-листа (открытие)",
        'text': "Спасибо большое, чек-лист заполнен. Настоятельно рекомендую проверить ТК для гостей. Пожалуйста, не забудьте вернуться на закрытие смены.",
        'type': 'message'
    },
    {
        'id': 21,
        'title': "Подготовка к завтрашней смене",
        'text': "Добрый вечер, {name}! Давайте подготовим заведение к завтрашней смене! Убедитесь в чистоте зоны бариста. Приложите фото.",
        'type': 'photo'
    },
    {
        'id': 22,
        'title': "Проверить бар (вечер), часть 1",
        'text': "Убедитесь в чистоте бара. Сделайте фото от весов до бойлера.",
        'type': 'photo'
    },
    {
        'id': 23,
        'title': "Проверить бар (вечер), часть 2",
        'text': "Убедитесь в чистоте бара 2. Сделайте фото от бойлера до раковины.",
        'type': 'photo'
    },
    {
        'id': 24,
        'title': "Температура бОльшей витрины (вечер)",
        'text': "Сколько градусов температура первой (бОльшей) витрины. Напишите цифрами.",
        'type': 'text'
    },
    {
        'id': 25,
        'title': "Температура малой витрины (вечер)",
        'text': "Сколько градусов температура второй (малой) витрины. Напишите цифрами.",
        'type': 'text'
    },
    {
        'id': 26,
        'title': "Протереть бОльшую витрину (вечер)",
        'text': "Протрите первую (бОльшую) витрину, убедитесь в наличии всех позиций, также проверьте товарный вид. Сделайте фото.",
        'type': 'photo'
    },
    {
        'id': 27,
        'title': "Протереть малую витрину (вечер)",
        'text': "Протрите вторую (малую) витрину, убедитесь в наличии всех позиций, проверьте товарный вид. Сделайте фото.",
        'type': 'photo'
    },
    {
        'id': 28,
        'title': "Протереть сервировочный стол",
        'text': "Полностью уберите все с сервировочного стола, протрите его. Сделайте фото пустого стола.",
        'type': 'photo'
    },
    {
        'id': 29,
        'title': "Заполнить сервировочный стол",
        'text': "Заполните полностью сервировочный стол по стандарту. Приложите фото.",
        'type': 'photo'
    },
    {
        'id': 30,
        'title': "Убрать остатки хлеба/выпечки",
        'text': "Уберите остатки хлеба и выпечки, если есть таковые. Сделайте фото пустой витрины.",
        'type': 'photo'
    },
    {
        'id': 31,
        'title': "Проверить диван и мягкие стулья",
        'text': "Проверьте диван и 4 мягких стула на наличие волос и/или крошек! Протрите столы, убедитесь в том, что подстолья чистые, стулья стоят на своих местах. Сделайте фото зала.",
        'type': 'photo'
    },
    {
        'id': 32,
        'title': "Проверить ТК (туалет)",
        'text': "Проверьте ТК для гостей. Убедитесь, что техперсонал тщательно помыл унитаз и раковину. Мусорное ведро должно быть пустым. Салфетница должна быть наполнена салфетками, проверьте мыло в дозаторе, предметы личной гигиены должны быть на месте. Сделайте фото столешницы.",
        'type': 'photo'
    },
    {
        'id': 33,
        'title': "Проверить мойку для черной посуды",
        'text': "Проверьте, чтобы мойка для черной посуды была чистой. Сделайте фото.",
        'type': 'photo'
    },
    {
        'id': 34,
        'title': "Проверить мойку для белой посуды",
        'text': "Проверьте, чтобы мойка для белой посуды была чистой. Сделайте фото стенда с чистой посудой.",
        'type': 'photo'
    },
    {
        'id': 35,
        'title': "Выключить музыку",
        'text': "Выключите музыку, переоденьтесь, уходя выключите свет. Сделайте фото микшера.",
        'type': 'photo'
    },
    {
        'id': 36,
        'title': "Выключить вытяжку",
        'text': "Выключите вытяжку. Сделайте фото.",
        'type': 'photo'
    },
    {
        'id': 37,
        'title': "Выключить кондиционер",
        'text': "Выключите кондиционер. Сделайте фото.",
        'type': 'photo'
    },
    {
        'id': 38,
        'title': "Выключить свет и выйти",
        'text': "Выходя, выключите свет. Сделайте фото с улицы.",
        'type': 'photo'
    },
    {
        'id': 39,
        'title': "Указать, кто был в зале",
        'text': "Напишите текстом, кто был на смене в зале",
        'type': 'text'
    },
    {
        'id': 40,
        'title': "Указать, кто был на кухне",
        'text': "Напишите, кто был на смене на кухне",
        'type': 'text'
    },
    {
        'id': 41,
        'title': "Задачи из передачи смены (выполненные)",
        'text': "Какие задачи из передачи смены вы выполнили за сегодня? Напишите текстом",
        'type': 'text'
    },
    {
        'id': 42,
        'title': "Задачи для передачи смены (новые)",
        'text': "Какие задачи вы хотите обозначить при передаче смен? Продублируйте текст в общую группу",
        'type': 'text'
    },
    {
        'id': 43,
        'title': "Закрытие смены",
        'text': "Спасибо большое за смену! Смена закрыта!",
        'type': 'message'
    }
]

BARISTA_CHECKLIST_TASKS = [
    {
        'id': 1,
        'title': "Включить бойлер",
        'text': "Доброе утро, {name}! Добро пожаловать на работу! Пожалуйста, включите бойлер, сделайте фото.",
        'type': 'photo'
    },
    {
        'id': 2,
        'title': "Включить кофейное оборудование",
        'text': "Включите оборудование для приготовления кофе, приложите фото. Приготовьте кофе.",
        'type': 'photo'
    },
    {
        'id': 3,
        'title': "Завершение чек-листа (открытие)",
        'text': "Спасибо большое, чек-лист заполнен. Пожалуйста, не забудьте вернуться на закрытие смены.",
        'type': 'message'
    },
    {
        'id': 4,
        'title': "Бутылки домашнего компота",
        'text': "Сколько бутылок домашнего компота? Напишите цифрами.",
        'type': 'text'
    },
    {
        'id': 5,
        'title': "Дата приготовления компота",
        'text': "Когда был приготовлен компот? Напишите дату.",
        'type': 'text'
    },
    {
        'id': 6,
        'title': "Бутылки «Свекла/Черная смородина»",
        'text': "Сколько бутылок Свекла/Черная смородина? Напишите цифрами.",
        'type': 'text'
    },
    {
        'id': 7,
        'title': "Дата приготовления (Свекла/Черная смородина)",
        'text': "Когда был приготовлен Свекла/Черная смородина? Напишите дату.",
        'type': 'text'
    },
    {
        'id': 8,
        'title': "Бутылки «Шиповник/Яблоко»",
        'text': "Сколько бутылок Шиповник/Яблоко? Напишите цифрами.",
        'type': 'text'
    },
    {
        'id': 9,
        'title': "Дата приготовления (Шиповник/Яблоко)",
        'text': "Когда был приготовлен Шиповник/Яблоко? Напишите дату.",
        'type': 'text'
    },
    {
        'id': 10,
        'title': "Бутылки «Алоэ/Мед/Имбирь»",
        'text': "Сколько бутылок Алоэ/Мед/Имбирь? Напишите цифрами.",
        'type': 'text'
    },
    {
        'id': 11,
        'title': "Дата приготовления (Алоэ/Мед/Имбирь)",
        'text': "Когда был приготовлен Алоэ/Мед/Имбирь? Напишите дату.",
        'type': 'text'
    },
    {
        'id': 12,
        'title': "Бутылки «Облепиха/Апельсин»",
        'text': "Сколько бутылок Облепиха/Апельсин? Напишите цифрами.",
        'type': 'text'
    },
    {
        'id': 13,
        'title': "Дата приготовления (Облепиха/Апельсин)",
        'text': "Когда был приготовлен Облепиха/Апельсин? Напишите дату.",
        'type': 'text'
    },
    {
        'id': 14,
        'title': "Бутылки «Пуэр Вишня»",
        'text': "Сколько бутылок Пуэр Вишня? Напишите цифрами.",
        'type': 'text'
    },
    {
        'id': 15,
        'title': "Дата приготовления (Пуэр Вишня)",
        'text': "Когда был приготовлен Пуэр Вишня? Напишите дату.",
        'type': 'text'
    },
    {
        'id': 16,
        'title': "Бутылки Cold Brew",
        'text': "Сколько бутылок Cold Brew? Напишите цифрами.",
        'type': 'text'
    },
    {
        'id': 17,
        'title': "Дата приготовления Cold Brew",
        'text': "Когда был приготовлен Cold Brew? Напишите дату.",
        'type': 'text'
    },
    {
        'id': 18,
        'title': "Бутылки Морса",
        'text': "Сколько бутылок Морса? Напишите цифрами.",
        'type': 'text'
    },
    {
        'id': 19,
        'title': "Дата приготовления Морса",
        'text': "Когда был приготовлен Морс? Напишите дату.",
        'type': 'text'
    },
    {
        'id': 20,
        'title': "Бутылки Кумыс",
        'text': "Сколько бутылок Кумыс? Напишите цифрами.",
        'type': 'text'
    },
    {
        'id': 21,
        'title': "Количество кофе (в граммах)",
        'text': "Сколько кофе у нас в наличии? Напишите в граммах.",
        'type': 'text'
    },
    {
        'id': 22,
        'title': "Фильтры под батч",
        'text': "Сколько фильтров под батч у нас в наличии? Напишите примерное число в цифрах.",
        'type': 'text'
    },
    {
        'id': 23,
        'title': "Фильтры под воронку",
        'text': "Сколько фильтров под воронку у нас в наличии? Напишите примерное число в цифрах.",
        'type': 'text'
    },
    {
        'id': 24,
        'title': "Заготовки «Облепиха-яблоко»",
        'text': "На сколько порций нам хватит заготовок Облепиха-яблоко? Напишите примерное число в цифрах.",
        'type': 'text'
    },
    {
        'id': 25,
        'title': "Заготовки «Жасмин-лаванда»",
        'text': "На сколько порций нам хватит заготовок Жасмин-лаванда? Напишите примерное число в цифрах.",
        'type': 'text'
    },
    {
        'id': 26,
        'title': "Заготовки «Вишня-чили»",
        'text': "На сколько порций нам хватит заготовок Вишня-чили? Напишите примерное число в цифрах.",
        'type': 'text'
    },
    {
        'id': 27,
        'title': "Заготовки «Малина-эвкалипт»",
        'text': "На сколько порций нам хватит заготовок Малина-эвкалипт? Напишите примерное число в цифрах.",
        'type': 'text'
    },
    {
        'id': 28,
        'title': "Заготовки «Глинтвейн безалкогольный»",
        'text': "На сколько порций нам хватит заготовок Глинтвейн безалкогольный? Напишите примерное число в цифрах.",
        'type': 'text'
    },
    {
        'id': 29,
        'title': "Запасы Черного чая",
        'text': "Сколько примерно грамм у нас осталось Черный чай? Напишите примерное количество в цифрах.",
        'type': 'text'
    },
    {
        'id': 30,
        'title': "Запасы Зеленого чая",
        'text': "Сколько примерно грамм у нас осталось Зеленый чай? Напишите примерное количество в цифрах.",
        'type': 'text'
    },
    {
        'id': 31,
        'title': "Литры молока",
        'text': "Сколько литров молока у нас в наличии?",
        'type': 'text'
    },
    {
        'id': 32,
        'title': "Выключить бойлер",
        'text': "Выключите бойлер, уберите рабочую поверхность. Сделайте фото.",
        'type': 'photo'
    },
    {
        'id': 33,
        'title': "Выключить кофейное оборудование",
        'text': "Выключите оборудование для приготовления кофе, уберите рабочее место. Приложите фото.",
        'type': 'photo'
    },
    {
        'id': 34,
        'title': "Выкинуть мусор на баре",
        'text': "Выкиньте мусор из мусорного ведра на баре. Вставьте новый пакет. Приложите фото.",
        'type': 'photo'
    },
    {
        'id': 35,
        'title': "Выкинуть мусор на кухне",
        'text': "Выкиньте мусор из мусорного ведра на кухне. Вставьте новый пакет. Приложите фото.",
        'type': 'photo'
    },
    {
        'id': 36,
        'title': "Выполненные задачи",
        'text': "Какие задачи, которые вам передавали по смене, вы выполнили? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 37,
        'title': "Задачи для передачи смены",
        'text': "Какие ключевые задачи вы хотите передать по смене? Напишите текстом, продублируйте в общую группу.",
        'type': 'text'
    },
    {
        'id': 38,
        'title': "Завершение смены (бариста)",
        'text': "Спасибо большое за смену! Смена закрыта!",
        'type': 'message'
    }
]

COOK_CHECKLIST_TASKS = [
    {
        'id': 1,
        'title': "Позиция повара",
        'text': "Доброе утро, {name}! Добро пожаловать на работу! На какой позиции преимущественно вы будете сегодня работать: повар горячего цеха, повар холодного цеха, повар универсал, лепка. Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 2,
        'title': "План задач на день",
        'text': "Какие задачи вы планируете сегодня выполнить? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 3,
        'title': "Окончание чек-листа (открытие)",
        'text': "Чек-лист открытия смены закончен. Для закрытия смены нажмите на кнопку ниже.",
        'type': 'message'
    },
    {
        'id': 4,
        'title': "Выполненные задачи",
        'text': "Какие задачи вы сегодня выполнили? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 5,
        'title': "Задачи на завтра",
        'text': "Передайте по смене список задач на завтра. Напишите текстом здесь, продублируйте в общую группу.",
        'type': 'text'
    },
    {
        'id': 6,
        'title': "Завершение смены (повар)",
        'text': "Спасибо большое за смену! Смена закрыта!",
        'type': 'message'
    }
]

CHEF_CHECKLIST_TASKS = [
    {
        'id': 1,
        'title': "Приветствие",
        'text': "Доброе утро, {name}! Добро пожаловать на работу!",
        'type': 'text'
    },
    {
        'id': 2,
        'title': "План задач на день",
        'text': "Какие задачи вы планируете сегодня выполнить? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 3,
        'title': "Окончание чек-листа открытия",
        'text': "Чек-лист открытия смены закончен. Для закрытия смены нажмите на кнопку ниже.",
        'type': 'message'
    },
    {
        'id': 4,
        'title': "Равиоли с гусем",
        'text': "Сколько порций «Равиоли с гусем» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 5,
        'title': "Чизбургер",
        'text': "Сколько порций «Чизбургер» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 6,
        'title': "Гречотто",
        'text': "Сколько порций «Гречотто» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 7,
        'title': "Филе курицы",
        'text': "Сколько порций «Филе курицы» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 8,
        'title': "Фланк стейк",
        'text': "Сколько порций «Фланк стейк» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 9,
        'title': "Сорпа с гусем",
        'text': "Сколько порций «Сорпа с гусем» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 10,
        'title': "Суп из курицы",
        'text': "Сколько порций «Суп из курицы» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 11,
        'title': "Говяжий хвост",
        'text': "Сколько порций «Говяжий хвост» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 12,
        'title': "Томленое ребро",
        'text': "Сколько порций «Томленое ребро» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 13,
        'title': "Куырдак",
        'text': "Сколько порций «Куырдак» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 14,
        'title': "Пюре с трюфелем",
        'text': "Сколько порций «Пюре с трюфелем» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 15,
        'title': "Брокколи",
        'text': "Сколько порций «Брокколи» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 16,
        'title': "Жаренные ньокки с шалфеем",
        'text': "Сколько порций «Жаренные ньокки с шалфеем» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 17,
        'title': "Вареники с картофелем и беконом",
        'text': "Сколько порций «Вареники с картофелем и беконом» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 18,
        'title': "Вареники с грибным соусом",
        'text': "Сколько порций «Вареники с картофелем и грибным соусом» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 19,
        'title': "Завтрак с краковской",
        'text': "Сколько порций «Завтрак с краковской» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 20,
        'title': "Завтрак с тушенкой",
        'text': "Сколько порций «Завтрак с тушенкой» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 21,
        'title': "Крок мадам",
        'text': "Сколько порций «Крок мадам» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 22,
        'title': "Завтрак с лососем",
        'text': "Сколько порций «Завтрак с лососем» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 23,
        'title': "Завтрак с сосисками",
        'text': "Сколько порций «Завтрак с молочными сосисками» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 24,
        'title': "Драники",
        'text': "Сколько порций «Драники» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 25,
        'title': "Шакшука",
        'text': "Сколько порций «Шакшука» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 26,
        'title': "Овсяная каша",
        'text': "Сколько порций «Овсяная каша» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 27,
        'title': "Рисовая каша",
        'text': "Сколько порций «Рисовая каша» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 28,
        'title': "Сырники со сгущенкой",
        'text': "Сколько порций «Сырники со сгущенкой» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 29,
        'title': "Сырники с мятной вишней",
        'text': "Сколько порций «Сырники с мятной вишней» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 30,
        'title': "Ремесленный хлеб",
        'text': "Сколько порций «Ремесленный хлеб» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 31,
        'title': "Моцарелла",
        'text': "Сколько порций «Моцарелла» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 32,
        'title': "Салат из корнеплодов",
        'text': "Сколько порций «Салат из корнеплодов» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 33,
        'title': "Хумус",
        'text': "Сколько порций «Хумус» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 34,
        'title': "Вителло тоннато",
        'text': "Сколько порций «Вителло тоннато» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 35,
        'title': "Мясная тарелка",
        'text': "Сколько порций «Мясная тарелка» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 36,
        'title': "Фокачча с мортаделлой",
        'text': "Сколько порций «Фокачча с мортаделлой» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 37,
        'title': "Фокачча с моцареллой",
        'text': "Сколько порций «Фокачча с моцареллой» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 38,
        'title': "Фокачча с салями",
        'text': "Сколько порций «Фокачча с салями» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 39,
        'title': "Панини с курицей",
        'text': "Сколько порций «Панини с курицей» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 40,
        'title': "Панини с ветчиной",
        'text': "Сколько порций «Панини с ветчиной» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 41,
        'title': "Панини с грибами",
        'text': "Сколько порций «Панини с грибами» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 42,
        'title': "Ньокки с шампиньонами",
        'text': "Сколько порций «Ньокки с шампиньонами» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 43,
        'title': "Ньокки с арабьята",
        'text': "Сколько порций «Ньокки с арабьята» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 44,
        'title': "Фетучини с песто",
        'text': "Сколько порций «Фетучини с песто» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 45,
        'title': "Папарделле с грибами",
        'text': "Сколько порций «Папарделле с грибами» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 46,
        'title': "Папарделле с гусем",
        'text': "Сколько порций «Папарделле с гусем» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 47,
        'title': "Гарганелли-карбонара",
        'text': "Сколько порций «Гарганелли-карбонара» в наличии? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 48,
        'title': "Выполненные задачи",
        'text': "Какие задачи вы сегодня выполнили? Напишите текстом.",
        'type': 'text'
    },
    {
        'id': 49,
        'title': "Задачи на завтра",
        'text': "Передайте по смене список задач на завтра. Напишите текстом здесь, продублируйте в общую группу.",
        'type': 'text'
    },
    {
        'id': 50,
        'title': "Кто был на смене",
        'text': "Кто сегодня был на смене? Пропишите текстом.",
        'type': 'text'
    },
    {
        'id': 51,
        'title': "Закрытие смены",
        'text': "Спасибо большое за смену! Смена закрыта!",
        'type': 'message'
    }
]


FORUM_CHAT_ID = -1002491476947

TOPIC_IDS = {
    'manager': 4,
    'barista': 3,
    'chef': 6,
    'cook': 5
}

def create_table():
    try:
        conn = sqlite3.connect('checklist.db')
        cursor = conn.cursor()

        # Создание таблицы для менеджера
        task_fields_list = []
        for task in CHECKLIST_TASKS:
            task_id = task['id']
            if task['type'] == 'photo':
                task_fields_list.append(f'photo_task{task_id} BLOB')
                task_fields_list.append(f'time_task{task_id} TEXT')
            elif task['type'] == 'text':
                task_fields_list.append(f'text_task{task_id} TEXT')
                task_fields_list.append(f'time_task{task_id} TEXT')
            # Если тип 'message', поля не нужны

        task_fields = ',\n                '.join(task_fields_list)

        sql_query = f'''
            CREATE TABLE IF NOT EXISTS manager (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                date TEXT,
                user_id INTEGER,
                name TEXT,
                {task_fields}
            )
        '''

        cursor.execute(sql_query)
        conn.commit()
        logger.info("Таблица для менеджера успешно создана или уже существует.")

        # Создание таблицы для бариста
        barista_task_fields_list = []
        for task in BARISTA_CHECKLIST_TASKS:
            task_id = task['id']
            if task['type'] == 'photo':
                barista_task_fields_list.append(f'photo_task{task_id} BLOB')
                barista_task_fields_list.append(f'time_task{task_id} TEXT')
            elif task['type'] == 'text':
                barista_task_fields_list.append(f'text_task{task_id} TEXT')
                barista_task_fields_list.append(f'time_task{task_id} TEXT')
            # Если тип 'message', поля не нужны

        barista_task_fields = ',\n                '.join(barista_task_fields_list)

        barista_sql_query = f'''
            CREATE TABLE IF NOT EXISTS barista (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                date TEXT,
                user_id INTEGER,
                name TEXT,
                {barista_task_fields}
            )
        '''

        cursor.execute(barista_sql_query)
        conn.commit()
        logger.info("Таблица для бариста успешно создана или уже существует.")

        # Создание таблицы для повара
        cook_task_fields_list = []
        for task in COOK_CHECKLIST_TASKS:
            task_id = task['id']
            if task['type'] == 'photo':
                cook_task_fields_list.append(f'photo_task{task_id} BLOB')
                cook_task_fields_list.append(f'time_task{task_id} TEXT')
            elif task['type'] == 'text':
                cook_task_fields_list.append(f'text_task{task_id} TEXT')
                cook_task_fields_list.append(f'time_task{task_id} TEXT')
            # Если тип 'message', поля не нужны

        cook_task_fields = ',\n                '.join(cook_task_fields_list)

        cook_sql_query = f'''
                    CREATE TABLE IF NOT EXISTS cook (
                        id INTEGER PRIMARY KEY AUTOINCREMENT,
                        date TEXT,
                        user_id INTEGER,
                        name TEXT,
                        {cook_task_fields}
                    )
                '''

        cursor.execute(cook_sql_query)
        conn.commit()
        logger.info("Таблица для повара успешно создана или уже существует.")

        # Создание таблицы для шефа
        chef_task_fields_list = []
        for task in CHEF_CHECKLIST_TASKS:
            task_id = task['id']
            if task['type'] == 'photo':
                chef_task_fields_list.append(f'photo_task{task_id} BLOB')
                chef_task_fields_list.append(f'time_task{task_id} TEXT')
            elif task['type'] == 'text':
                chef_task_fields_list.append(f'text_task{task_id} TEXT')
                chef_task_fields_list.append(f'time_task{task_id} TEXT')
            # Если тип 'message', поля не нужны

        chef_task_fields = ',\n                '.join(chef_task_fields_list)

        chef_sql_query = f'''
                    CREATE TABLE IF NOT EXISTS chef (
                        id INTEGER PRIMARY KEY AUTOINCREMENT,
                        date TEXT,
                        user_id INTEGER,
                        name TEXT,
                        {chef_task_fields}
                    )
                '''

        cursor.execute(chef_sql_query)
        conn.commit()
        logger.info("Таблица для шефа успешно создана или уже существует.")
    except Exception as e:
        logger.error(f"Ошибка в функции create_table: {e}")
        print(f"Ошибка в функции create_table: {e}")
    finally:
        conn.close()

# Стартовая функция
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE) -> int:
    logger.info("Начало работы с пользователем")
    # Создаем кнопки для выбора профессии
    keyboard = [
        [InlineKeyboardButton("Менеджер", callback_data='profession_manager')],
        [InlineKeyboardButton("Бармен", callback_data='profession_bartender')],
        [InlineKeyboardButton("Повар", callback_data='profession_cook')],
        [InlineKeyboardButton("Шеф-повар", callback_data='profession_chef')],
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)

    await update.message.reply_text(
        "Здравствуйте! Пожалуйста, выберите вашу должность:",
        reply_markup=reply_markup
    )
    return SELECT_PROFESSION  # Переходим к состоянию выбора профессии

# Обработчик выбора профессии
async def select_profession(update: Update, context: ContextTypes.DEFAULT_TYPE) -> int:
    query = update.callback_query
    await query.answer()
    profession = query.data
    logger.info(f"Профессия выбрана: {profession}")

    if profession == 'profession_manager':
        # Сохраняем выбранную профессию в user_data
        context.user_data['profession'] = 'manager'

        # Отправляем кнопку "Открыть смену"
        keyboard = [
            [InlineKeyboardButton("Открыть смену", callback_data='open_shift')]
        ]
        reply_markup = InlineKeyboardMarkup(keyboard)

        await query.message.reply_text(
            "Добро пожаловать на работу, менеджер! Нажмите 'Открыть смену', чтобы начать.",
            reply_markup=reply_markup
        )
        return GET_NAME  # Переходим к состоянию ввода имени

    elif profession == 'profession_bartender':
        # Сохраняем выбранную профессию в user_data
        context.user_data['profession'] = 'barista'

        # Отправляем кнопку "Открыть смену"
        keyboard = [
            [InlineKeyboardButton("Открыть смену", callback_data='open_shift')]
        ]
        reply_markup = InlineKeyboardMarkup(keyboard)

        await query.message.reply_text(
            "Добро пожаловать на работу, бариста! Нажмите 'Открыть смену', чтобы начать.",
            reply_markup=reply_markup
        )
        return GET_NAME  # Переходим к состоянию ввода имени
    elif profession == 'profession_cook':
        # Сохраняем выбранную профессию в user_data
        context.user_data['profession'] = 'cook'

        # Отправляем кнопку "Открыть смену"
        keyboard = [
            [InlineKeyboardButton("Открыть смену", callback_data='open_shift')]
        ]
        reply_markup = InlineKeyboardMarkup(keyboard)

        await query.message.reply_text(
            "Добро пожаловать на работу, повар! Нажмите 'Открыть смену', чтобы начать.",
            reply_markup=reply_markup
        )
        return GET_NAME  # Переходим к состоянию ввода имени
    elif profession == 'profession_chef':
        # Сохраняем выбранную профессию в user_data
        context.user_data['profession'] = 'chef'

        # Отправляем кнопку "Открыть смену"
        keyboard = [
            [InlineKeyboardButton("Открыть смену", callback_data='open_shift')]
        ]
        reply_markup = InlineKeyboardMarkup(keyboard)

        await query.message.reply_text(
            "Добро пожаловать на работу, шеф! Нажмите 'Открыть смену', чтобы начать.",
            reply_markup=reply_markup
        )
        return GET_NAME  # Переходим к состоянию ввода имени
    else:
        await query.message.reply_text("Извините, эта профессия пока не реализована.")
        return ConversationHandler.END

# Открытие смены
async def open_shift(update: Update, context: ContextTypes.DEFAULT_TYPE) -> int:
    query = update.callback_query
    await query.answer()
    await query.message.reply_text("Пожалуйста, введите ваше имя.")
    return GET_NAME  # Переходим к следующему состоянию

async def start_close_shift(update: Update, context: ContextTypes.DEFAULT_TYPE) -> int:
    query = update.callback_query
    await query.answer()
    profession = context.user_data.get('profession')

    await query.message.reply_text("Вы начали процесс закрытия смены.")

    if profession == 'manager':
        # Начинаем с задачи id == 21 для менеджера
        tasks = CHECKLIST_TASKS
        for i, task in enumerate(tasks):
            if task['id'] == 21:
                context.user_data['task_index'] = i
                break
    elif profession == 'barista':
        # Начинаем с задачи id == 4 для бариста
        tasks = BARISTA_CHECKLIST_TASKS
        for i, task in enumerate(tasks):
            if task['id'] == 4:
                context.user_data['task_index'] = i
                break
    elif profession == 'cook':
        # Начинаем с задачи id == 4 для бариста
        tasks = COOK_CHECKLIST_TASKS
        for i, task in enumerate(tasks):
            if task['id'] == 4:
                context.user_data['task_index'] = i
                break
    elif profession == 'chef':
        # Начинаем с задачи id == 4 для бариста
        tasks = CHEF_CHECKLIST_TASKS
        for i, task in enumerate(tasks):
            if task['id'] == 4:
                context.user_data['task_index'] = i
                break
    else:
        await query.message.reply_text("Неизвестная профессия.")
        return ConversationHandler.END

    await send_next_task(update, context)
    return TASKS
# Сохраняем имя и переходим к задачам
async def get_name(update: Update, context: ContextTypes.DEFAULT_TYPE) -> int:
    name = update.message.text
    context.user_data['name'] = name
    user_id = update.effective_user.id
    date = datetime.now().strftime('%d.%m')
    profession = context.user_data.get('profession')

    # Создаём запись в базе данных
    try:
        conn = sqlite3.connect('checklist.db')
        cursor = conn.cursor()

        if profession == 'manager':
            cursor.execute('''
                INSERT INTO manager (date, user_id, name)
                VALUES (?, ?, ?)
            ''', (date, user_id, name))
            conn.commit()
            db_id = cursor.lastrowid
            context.user_data['db_id'] = db_id
            logger.info(f"Создана запись в БД manager с id {db_id}")

        elif profession == 'barista':
            cursor.execute('''
                INSERT INTO barista (date, user_id, name)
                VALUES (?, ?, ?)
            ''', (date, user_id, name))
            conn.commit()
            db_id = cursor.lastrowid
            context.user_data['db_id'] = db_id
            logger.info(f"Создана запись в БД barista_zala с id {db_id}")

        elif profession == 'cook':
            cursor.execute('''
                INSERT INTO cook (date, user_id, name)
                VALUES (?, ?, ?)
            ''', (date, user_id, name))
            conn.commit()
            db_id = cursor.lastrowid
            context.user_data['db_id'] = db_id
            logger.info(f"Создана запись в БД cook с id {db_id}")

        elif profession == 'chef':
            cursor.execute('''
                INSERT INTO chef (date, user_id, name)
                VALUES (?, ?, ?)
            ''', (date, user_id, name))
            conn.commit()
            db_id = cursor.lastrowid
            context.user_data['db_id'] = db_id
            logger.info(f"Создана запись в БД chef с id {db_id}")
        else:
            await update.message.reply_text("Неизвестная профессия.")
            return ConversationHandler.END

    except Exception as e:
        logger.error(f"Ошибка при создании записи в БД: {e}")
        await update.message.reply_text("Произошла ошибка при сохранении данных. Попробуйте позже.")
        return ConversationHandler.END
    finally:
        conn.close()

    # Инициализируем task_index
    context.user_data['task_index'] = 0  # Начинаем с первой задачи

    await update.message.reply_text(f"Приятно познакомиться, {name}!")
    # Отправляем первую задачу
    await send_next_task(update, context)
    return TASKS  # Переходим к следующему состоянию

async def send_next_task(update: Update, context: ContextTypes.DEFAULT_TYPE) -> int:
    task_index = context.user_data['task_index']
    name = context.user_data.get('name', '')
    profession = context.user_data.get('profession')

    # Выбираем соответствующий список задач
    if profession == 'manager':
        tasks = CHECKLIST_TASKS
    elif profession == 'barista':
        tasks = BARISTA_CHECKLIST_TASKS
    elif profession == 'cook':
        tasks = COOK_CHECKLIST_TASKS
    elif profession == 'chef':
        tasks = CHEF_CHECKLIST_TASKS
    else:
        await update.effective_message.reply_text("Неизвестная профессия.")
        return ConversationHandler.END

    # Проверяем, есть ли ещё задачи
    if task_index >= len(tasks):
        await update.effective_message.reply_text("Все задачи выполнены. Спасибо за работу!")
        return ConversationHandler.END

    task = tasks[task_index]
    task_text = task['text'].format(name=name)
    task_type = task['type']
    task_id = task['id']

    # Сохраняем текущий тип и ID задачи для обработки ответа
    context.user_data['current_task_type'] = task_type
    context.user_data['current_task_id'] = task_id

    # Обработка задач типа 'message'
    if task_type == 'message':
        await update.effective_message.reply_text(task_text)

        # Исключения для менеджера
        if profession == 'manager':
            if task_id == 20:
                # После завершения открытия смены
                keyboard = [
                    [InlineKeyboardButton("Закрыть смену", callback_data='close_shift')]
                ]
                reply_markup = InlineKeyboardMarkup(keyboard)
                await update.effective_message.reply_text(
                    "Вы можете закрыть смену, нажав на кнопку ниже.",
                    reply_markup=reply_markup
                )
                return TASKS  # Остаёмся в текущем состоянии
            elif task_id == 43:
                # После закрытия смены
                keyboard = [
                    [InlineKeyboardButton("Открыть смену", callback_data='open_shift')]
                ]
                reply_markup = InlineKeyboardMarkup(keyboard)
                await update.effective_message.reply_text(
                    "Вы можете открыть новую смену, нажав на кнопку ниже.",
                    reply_markup=reply_markup
                )
                context.user_data.pop('task_index', None)
                context.user_data.pop('db_id', None)
                context.user_data.pop('current_task_type', None)
                context.user_data.pop('current_task_id', None)
                return GET_NAME  # Возвращаемся к состоянию ввода имени
            else:
                # Переходим к следующей задаче
                context.user_data['task_index'] += 1
                await send_next_task(update, context)
                return TASKS

        # Исключения для бариста
        elif profession == 'barista':
            if task_id == 3:
                # После завершения открытия смены
                keyboard = [
                    [InlineKeyboardButton("Закрыть смену", callback_data='close_shift')]
                ]
                reply_markup = InlineKeyboardMarkup(keyboard)
                await update.effective_message.reply_text(
                    "Вы можете закрыть смену, нажав на кнопку ниже.",
                    reply_markup=reply_markup
                )
                return TASKS
            elif task_id == 38:
                # После закрытия смены
                keyboard = [
                    [InlineKeyboardButton("Открыть смену", callback_data='open_shift')]
                ]
                reply_markup = InlineKeyboardMarkup(keyboard)
                await update.effective_message.reply_text(
                    "Вы можете открыть новую смену, нажав на кнопку ниже.",
                    reply_markup=reply_markup
                )
                context.user_data.pop('task_index', None)
                context.user_data.pop('db_id', None)
                context.user_data.pop('current_task_type', None)
                context.user_data.pop('current_task_id', None)
                return GET_NAME
            else:
                # Переходим к следующей задаче
                context.user_data['task_index'] += 1
                await send_next_task(update, context)
                return TASKS

        elif profession == 'cook':
            if task_id == 3:
                # После завершения открытия смены
                keyboard = [
                    [InlineKeyboardButton("Закрыть смену", callback_data='close_shift')]
                ]
                reply_markup = InlineKeyboardMarkup(keyboard)
                await update.effective_message.reply_text(
                    "Вы можете закрыть смену, нажав на кнопку ниже.",
                    reply_markup=reply_markup
                )
                return TASKS
            elif task_id == 6:
                # После закрытия смены
                keyboard = [
                    [InlineKeyboardButton("Открыть смену", callback_data='open_shift')]
                ]
                reply_markup = InlineKeyboardMarkup(keyboard)
                await update.effective_message.reply_text(
                    "Вы можете открыть новую смену, нажав на кнопку ниже.",
                    reply_markup=reply_markup
                )
                context.user_data.pop('task_index', None)
                context.user_data.pop('db_id', None)
                context.user_data.pop('current_task_type', None)
                context.user_data.pop('current_task_id', None)
                return GET_NAME
            else:
                # Переходим к следующей задаче
                context.user_data['task_index'] += 1
                await send_next_task(update, context)
                return TASKS

        elif profession == 'chef':
            if task_id == 3:
                # После завершения открытия смены
                keyboard = [
                    [InlineKeyboardButton("Закрыть смену", callback_data='close_shift')]
                ]
                reply_markup = InlineKeyboardMarkup(keyboard)
                await update.effective_message.reply_text(
                    "Вы можете закрыть смену, нажав на кнопку ниже.",
                    reply_markup=reply_markup
                )
                return TASKS
            elif task_id == 59:
                # После закрытия смены
                keyboard = [
                    [InlineKeyboardButton("Открыть смену", callback_data='open_shift')]
                ]
                reply_markup = InlineKeyboardMarkup(keyboard)
                await update.effective_message.reply_text(
                    "Вы можете открыть новую смену, нажав на кнопку ниже.",
                    reply_markup=reply_markup
                )
                context.user_data.pop('task_index', None)
                context.user_data.pop('db_id', None)
                context.user_data.pop('current_task_type', None)
                context.user_data.pop('current_task_id', None)
                return GET_NAME
            else:
                # Переходим к следующей задаче
                context.user_data['task_index'] += 1
                await send_next_task(update, context)
                return TASKS
    # Обработка задач других типов ('photo', 'text')
    else:
        await update.effective_message.reply_text(task_text)
        return TASKS  # Остаёмся в состоянии выполнения задач

async def handle_task_response(update: Update, context: ContextTypes.DEFAULT_TYPE) -> int:
    # Проверяем, является ли это нажатие на кнопку
    if update.callback_query:
        query = update.callback_query
        await query.answer()
        if query.data == 'close_shift':
            # Начинаем процесс закрытия смены
            return await start_close_shift(update, context)
        elif query.data == 'open_shift':
            # Начинаем процесс открытия новой смены
            await query.message.reply_text("Пожалуйста, введите ваше имя.")
            return GET_NAME
        else:
            await query.message.reply_text("Неизвестная команда.")
            return TASKS

    # Обработка ответов на задачи
    task_index = context.user_data.get('task_index', 0)
    profession = context.user_data.get('profession')
    name = context.user_data.get('name', 'Безымянный')
    topic_id = TOPIC_IDS.get(profession, 0)  # если почему-то нет profession — будет 0 (основная ветка)

    if profession == 'manager':
        tasks = CHECKLIST_TASKS
        table_name = 'manager'
    elif profession == 'barista':
        tasks = BARISTA_CHECKLIST_TASKS
        table_name = 'barista'
    elif profession == 'cook':
        tasks = COOK_CHECKLIST_TASKS
        table_name = 'cook'
    elif profession == 'chef':
        tasks = CHEF_CHECKLIST_TASKS
        table_name = 'chef'
    else:
        await update.message.reply_text("Неизвестная профессия.")
        return ConversationHandler.END

    if task_index >= len(tasks):
        await update.message.reply_text("Все задачи выполнены. Спасибо за работу!")
        return ConversationHandler.END

    task = tasks[task_index]
    task_id = task['id']
    task_type = context.user_data['current_task_type']
    title = task.get('title', f"task #{task_id}")
    db_id = context.user_data['db_id']

    # Если тип задачи 'message', мы не ожидаем ответа, но проверим на случай, если пользователь отправил что-то
    if task_type == 'message':
        # Игнорируем сообщение пользователя и переходим к следующей задаче
        context.user_data['task_index'] += 1
        await send_next_task(update, context)
        return TASKS

    try:

        conn = sqlite3.connect('checklist.db')
        cursor = conn.cursor()

        if task_type == 'photo':
            if update.message.photo:
                photo_file = await update.message.photo[-1].get_file()
                file_id = photo_file.file_id
                photo_binary = await photo_file.download_as_bytearray()
                time_now = datetime.now().strftime('%H:%M')

                # Обновляем соответствующие поля в базе данных
                cursor.execute(f'''
                    UPDATE {table_name}
                    SET photo_task{task_id} = ?, time_task{task_id} = ?
                    WHERE id = ?
                ''', (photo_binary, time_now, db_id))
                conn.commit()
                await context.bot.send_photo(
                    chat_id=FORUM_CHAT_ID,
                    message_thread_id=topic_id,
                    photo=file_id,
                    caption=f"Фото от {name} ({profession}), {title}"
                )
                await update.message.reply_text("Фото получено.")
            else:
                await update.message.reply_text("Пожалуйста, отправьте фото.")
                return TASKS  # Остаемся в текущем состоянии

        elif task_type == 'text':
            if update.message.text:
                text_response = update.message.text
                time_now = datetime.now().strftime('%H:%M')

                # Обновляем соответствующее поле в базе данных
                cursor.execute(f'''
                    UPDATE {table_name}
                    SET text_task{task_id} = ?, time_task{task_id} = ?
                    WHERE id = ?
                ''', (text_response, time_now, db_id))
                conn.commit()

                await context.bot.send_message(
                    chat_id=FORUM_CHAT_ID,
                    message_thread_id=topic_id,
                    text=f"*{name}* ({profession}), *{title}*:\n{text_response}",
                    parse_mode="Markdown"
                )
                
                await update.message.reply_text("Ответ получен.")
            else:
                await update.message.reply_text("Пожалуйста, отправьте текстовый ответ.")
                return TASKS  # Остаемся в текущем состоянии

    except Exception as e:
        logger.error(f"Ошибка при обработке ответа: {e}")
        await update.message.reply_text("Произошла ошибка при сохранении вашего ответа. Попробуйте снова.")
        return TASKS
    finally:
        conn.close()

    # Переходим к следующей задаче
    context.user_data['task_index'] += 1
    await send_next_task(update, context)
    return TASKS

async def reset(update: Update, context: ContextTypes.DEFAULT_TYPE) -> int:
    # Очищаем все данные, хранящиеся для текущего пользователя
    context.user_data.clear()

    # Предлагаем пользователю начать все заново
    await update.message.reply_text("Все сброшено. Пожалуйста, введите /start, чтобы начать сначала.")
    return ConversationHandler.END

async def info(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    await update.message.reply_text(
        "Этот бот — чеклист для сотрудников Harvest.\n"
        "Если бот перестал отвечать, вызовите команду /reset, чтобы сбросить процесс и начать заново."
    )


def main():
    create_table()
    application = Application.builder().token("7172280603:AAGu9LXCQfZfrsqX8z2d-wcpQh_Ef5x77R4").build()

    conv_handler = ConversationHandler(
        entry_points=[CommandHandler('start', start)],
        states={
            SELECT_PROFESSION: [
                CallbackQueryHandler(select_profession, pattern='^profession_.*$')
            ],
            GET_NAME: [
                CallbackQueryHandler(open_shift, pattern='^open_shift$'),
                MessageHandler(filters.TEXT & ~filters.COMMAND, get_name)
            ],
            TASKS: [
                CallbackQueryHandler(start_close_shift, pattern='^close_shift$'),
                CallbackQueryHandler(open_shift, pattern='^open_shift$'),
                MessageHandler(filters.ALL & ~filters.COMMAND, handle_task_response)
            ],
        },
        fallbacks=[CommandHandler('reset', reset)],
    )

    application.add_handler(CommandHandler('info', info))
    application.add_handler(conv_handler)
    application.run_polling()

if __name__ == '__main__':
    create_table()
    main()
