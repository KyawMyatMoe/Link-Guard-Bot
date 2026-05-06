import re
import logging
import os
from telegram import Update, MessageEntity
from telegram.ext import Application, MessageHandler, filters, ContextTypes

BOT_TOKEN = "8737788615:AAHHNkh5EpSyi-B9LwLY_Ae-3P3e8nwO1IA"
WARNING_MESSAGE = "⚠️ Unknown Link များ ပို့ခွင့်မရှိပါ။ သင့် message ကို ဖျက်လိုက်ပါပြီ။"

logging.basicConfig(format="%(asctime)s - %(name)s - %(levelname)s - %(message)s", level=logging.INFO)
logger = logging.getLogger(__name__)

LINK_REGEX = re.compile(r"\b(?:https?://|t\.me/|www\.)\S+\b", re.IGNORECASE)

async def check_for_links(text):
    return bool(LINK_REGEX.search(text))

async def handle_message(update, context):
    message = update.effective_message
    chat = update.effective_chat
    if not message or not chat or not message.text:
        return
    if chat.type in ["group", "supergroup"]:
        chat_admins = await chat.get_administrators()
        admin_ids = {admin.user.id for admin in chat_admins}
        if message.from_user.id in admin_ids:
            return
    has_link = False
    if message.entities:
        for entity in message.entities:
            if entity.type in [MessageEntity.URL, MessageEntity.TEXT_LINK]:
                has_link = True
                break
    if not has_link:
        has_link = await check_for_links(message.text)
    if has_link:
        try:
            await context.bot.send_message(chat_id=chat.id, text=WARNING_MESSAGE, reply_to_message_id=message.message_id)
            await message.delete()
        except Exception as e:
            logger.error(f"Error: {e}")

def main():
    application = Application.builder().token(BOT_TOKEN).build()
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))
    application.run_polling(allowed_updates=Update.ALL_TYPES)

if __name__ == "__main__":
    main()