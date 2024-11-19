from telegram import Update, ChatPermissions
from telegram.ext import Application, MessageHandler, filters, ContextTypes

# Токен вашего бота
TOKEN = "7968470471:AAFq0lsdALr8qUokMEs1ZysbHkh_VpaNrGA"

# Список запрещенных слов или фраз
TRIGGER_WORDS = ["@TurnedV", "слив", "скрипты вашего", "слил"]

# Функция для удаления всех сообщений пользователя
async def delete_all_messages(user_id: int, chat_id: int, context: ContextTypes.DEFAULT_TYPE):
    async for message in context.bot.get_chat_history(chat_id):
        if message.from_user and message.from_user.id == user_id:
            try:
                await context.bot.delete_message(chat_id=chat_id, message_id=message.message_id)
            except Exception as e:
                print(f"Ошибка при удалении сообщения: {e}")

# Обработчик сообщений
async def delete_and_ban(update: Update, context: ContextTypes.DEFAULT_TYPE):
    message = update.message
    user = update.effective_user
    chat = update.effective_chat

    # Проверяем текст сообщения на наличие запрещенных слов
    if message and message.text and any(word in message.text.lower() for word in TRIGGER_WORDS):
        try:
            # Удаляем сообщение
            await context.bot.delete_message(chat_id=chat.id, message_id=message.message_id)
            print(f"Удалено сообщение от пользователя {user.username}")

            # Удаляем все сообщения пользователя
            await delete_all_messages(user_id=user.id, chat_id=chat.id, context=context)

            # Блокируем пользователя
            await context.bot.restrict_chat_member(
                chat_id=chat.id,
                user_id=user.id,
                permissions=ChatPermissions(can_send_messages=False)
            )

            # Отправляем сообщение в чат
            await context.bot.send_message(
                chat_id=chat.id,
                text="Съебал нахуй с чата, сын шлюхи!"
            )
        except Exception as e:
            print(f"Ошибка: {e}")

# Основная функция для запуска бота
def main():
    # Инициализация приложения
    application = Application.builder().token(TOKEN).build()

    # Обработчик текстовых сообщений
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, delete_and_ban))

    # Запуск бота
    print("Бот запущен...")
    application.run_polling()

if __name__ == "__main__":
    main()

