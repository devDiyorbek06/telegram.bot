# telegram.bot
telegram bot yaratish uzim uchun
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ConversationHandler
import phonenumbers
from phonenumbers import carrier, geocoder, timezone

ASK_NAME, ASK_SURNAME, ASK_LOCATION, ASK_PHONE = range(4)

async def start(update: Update, context) -> int:
    await update.message.reply_text("Assalomu alaykum, Yaxyayev Diyorbekning botiga xush kelibsiz! Ismingizni kiriting:")
    return ASK_NAME  # Keyingi bosqichga o'tish

async def ask_name(update: Update, context) -> int:
    context.user_data['name'] = update.message.text
    await update.message.reply_text("Familiyangizni kiriting:")
    return ASK_SURNAME  # Keyingi bosqichga o'tish

async def ask_surname(update: Update, context) -> int:
    context.user_data['surname'] = update.message.text
    await update.message.reply_text("Qayerdansiz:")
    return ASK_LOCATION  # Keyingi bosqichga o'tish

async def ask_location(update: Update, context) -> int:
    context.user_data['location'] = update.message.text
    await update.message.reply_text("Telefon raqamingizni kiriting:")
    return ASK_PHONE  # Keyingi bosqichga o'tish

def validate_phone_number(phone_number, region="UZ"):
    try:
        # Telefon raqamini analiz qilish
        parsed_number = phonenumbers.parse(phone_number, region)

        # Raqamning mumkin va to'g'ri ekanligini tekshirish
        if phonenumbers.is_possible_number(parsed_number) and phonenumbers.is_valid_number(parsed_number):
            # Qo'shimcha ma'lumotlar
            return {
                "valid": True,
                "operator": carrier.name_for_number(parsed_number, "en"),
                "location": geocoder.description_for_number(parsed_number, "en"),
                "timezones": timezone.time_zones_for_number(parsed_number)
            }
        else:
            return {"valid": False}
    except phonenumbers.NumberParseException:
        return {"valid": False}

async def ask_phone(update: Update, context) -> int:
    phone_number = update.message.text
    validation_result = validate_phone_number(phone_number)
    
    if validation_result["valid"]:
        context.user_data['phone'] = phone_number
        await update.message.reply_text(
            f"Katta rahmat! Ma'lumotlaringiz:\n"
            f"Ism: {context.user_data['name']}\n"
            f"Familiya: {context.user_data['surname']}\n"
            f"Qayerdan: {context.user_data['location']}\n"
            f"Telefon: {context.user_data['phone']}\n"
            f"Operator: {validation_result['operator']}\n"
            f"Joylashuv: {validation_result['location']}\n"
            f"Vaqt zonalari: {', '.join(validation_result['timezones'])}"
        )
    else:
        await update.message.reply_text("Telefon raqami noto'g'ri. Iltimos, qayta urinib ko'ring.")
        return ASK_PHONE  # Telefon raqamini qayta so'rash

    return ConversationHandler.END  # Suhbat tugashi

async def cancel(update: Update, context) -> int:
    await update.message.reply_text("Suhbat bekor qilindi.")
    return ConversationHandler.END

def main() -> None:
    # Botni sozlash
    app = Application.builder().token("7406956949:AAHFiFPX8YXSRPI512oWOq2esptBGKeHxyA").build()

    # Suhbatni boshqarish uchun ConversationHandler
    conv_handler = ConversationHandler(
        entry_points=[CommandHandler('start', start)],  # Suhbatni boshlash nuqtasi
        states={
            ASK_NAME: [MessageHandler(filters.TEXT & ~filters.COMMAND, ask_name)],
            ASK_SURNAME: [MessageHandler(filters.TEXT & ~filters.COMMAND, ask_surname)],
            ASK_LOCATION: [MessageHandler(filters.TEXT & ~filters.COMMAND, ask_location)],
            ASK_PHONE: [MessageHandler(filters.TEXT & ~filters.COMMAND, ask_phone)],
        },
        fallbacks=[CommandHandler('cancel', cancel)]  # Suhbatni bekor qilish
    )

    # Handlerni botga qo'shish
    app.add_handler(conv_handler)

    # Botni ishga tushirish
    print("Bot ishlamoqda...")
    app.run_polling()

if __name__ == '__main__':
    main()

def save_to_file(user_data):
    with open("user_data.txt", "a") as file:
        file.write(f"Ism: {user_data['name']}, Familiya: {user_data['surname']}, Qayerdan: {user_data['location']}, Telefon: {user_data['phone']}\n")

# Misol:
user_data = {
    'name': 'Diyorbek',
    'surname': 'Abdullaev',
    'location': 'Toshkent',
    'phone': '+998901234567'
}
save_to_file(user_data)
