from telegram import Update, ReplyKeyboardMarkup, KeyboardButton
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, ContextTypes, filters, ConversationHandler
NAME, EMAIL = range(2)
users = {}

main_keyboard = ReplyKeyboardMarkup(
    keyboard=[
        [KeyboardButton("Каталог"), KeyboardButton("Корзина")],
        [KeyboardButton("Промокоды"), KeyboardButton("Реф система")],
        [KeyboardButton("Личный кабинет")]
    ],
    resize_keyboard=True
)

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "Добро пожаловать в магазин одежды!\nВыберите действие:",
        reply_markup=main_keyboard,
    )

async def personal_area(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    if user_id in users:
        user = users[user_id]
        text = (f"Ваш профиль:\n"
                f"Имя: {user['name']}\n"
                f"Email: {user['email']}\n\n"
                f"Чтобы обновить данные, отправьте /register")
    else:
        text = "Вы не зарегистрированы.\nЧтобы пройти регистрацию, отправьте /register"
    await update.message.reply_text(text)

async def register_start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Пожалуйста, введите ваше имя:")
    return NAME

async def register_name(update: Update, context: ContextTypes.DEFAULT_TYPE):
    context.user_data['name'] = update.message.text.strip()
    await update.message.reply_text("Введите ваш email:")
    return EMAIL

async def register_email(update: Update, context: ContextTypes.DEFAULT_TYPE):
    email = update.message.text.strip()
    if "@" not in email or '.' not in email:
        await update.message.reply_text("Некорректный email. Попробуйте ещё раз:")
        return EMAIL

    context.user_data['email'] = email
    users[update.effective_user.id] = context.user_data.copy()
    await update.message.reply_text("Регистрация успешно завершена!", reply_markup=main_keyboard)
    return ConversationHandler.END

async def placeholders(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Пока тут ничего нет.")

async def unknown(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Неизвестная команда или сообщение. Используйте кнопки меню.")

async def cancel(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Регистрация отменена.", reply_markup=main_keyboard)
    return ConversationHandler.END

def main():
    TOKEN = ""
    app = ApplicationBuilder().token(TOKEN).build()

    conv_handler = ConversationHandler(
        entry_points=[CommandHandler('register', register_start)],
        states={
            NAME: [MessageHandler(filters.TEXT & ~filters.COMMAND, register_name)],
            EMAIL: [MessageHandler(filters.TEXT & ~filters.COMMAND, register_email)],
        },
        fallbacks=[CommandHandler('cancel', cancel)],
    )

    app.add_handler(CommandHandler("start", start))
    app.add_handler(conv_handler)
    app.add_handler(MessageHandler(filters.Regex('^Личный кабинет$'), personal_area))
    app.add_handler(MessageHandler(filters.Regex('^(Каталог|Корзина|Промокоды|Реф система)$'),placeholders))
    app.add_handler(MessageHandler(filters.COMMAND, unknown))
    app.add_handler(MessageHandler(filters.TEXT, unknown))

    print("Бот запущен...")
    app.run_polling()

if name == "__main__":
    main()
