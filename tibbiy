import logging
import re
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters
from telegram.ext import CallbackContext

# Telegram botni sozlash
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO)
logger = logging.getLogger(__name__)

# Maxsus belgilar va ularning maqsadi (tibbiy belgilar)
special_chars_dict = {
    r'\*': 'Asterisk (*) - ko\'pincha tibbiy ma\'lumotlarda qo\'llanadi (dori-darmon yoki simptomlar ro\'yxati)',
    r'\#': 'Hashtag (#) - kasallik yoki holatlarni tasvirlashda ishlatiladi',
    r'\@': 'At symbol (@) - ijtimoiy tarmoqlar yoki tibbiy mutaxassislarni belgilash uchun ishlatiladi',
    r'\$': 'Dollar sign ($) - dorilar yoki tibbiy xizmatlarning narxlarini ko\'rsatish uchun ishlatiladi',
    r'\%': 'Percent (%) - tibbiy foizlar, masalan, bemorning shifo olish foizi yoki test natijalari',
    r'\&': 'Ampersand (&) - "va" deb tarjima qilinadi, dori tarkibini birlashtirishda ishlatiladi (masalan, "Aspirin & Paracetamol")',
    r'\+': 'Plus sign (+) - qo\'shish, kombinatsiya yoki o\'zgarishlar belgilari uchun ishlatiladi',
    r'\!': 'Exclamation mark (!) - tibbiy ogohlantirishlar yoki muhim xabarlarni ko\'rsatish uchun ishlatiladi',
    r'\(': 'Left Parenthesis ( - tibbiy parametrlarni ajratishda ishlatiladi (masalan, "Xastalik (kasallik nomi)")',
    r'\)': 'Right Parenthesis ( ) - o\'zgaruvchilarni yopish yoki qo\'shimcha ma\'lumot berish uchun ishlatiladi',
    r'\[': 'Left Bracket ([) - ixtiyoriy tibbiy qiymatlar yoki test natijalarini ajratish uchun',
    r'\]': 'Right Bracket (]) - ixtiyoriy tibbiy qiymatlarni yopish uchun ishlatiladi',
    r'\{': 'Left Brace ({) - shartli yoki funktsional parametrlarni ajratishda ishlatiladi',
    r'\}': 'Right Brace (}) - shartli yoki funktsional parametrlarni yopishda ishlatiladi',
    r'°': 'Degree symbol (°) - tana harorati yoki boshqa o\'lchov birligi uchun ishlatiladi (masalan, 37°C)',
    r'≠': 'Not Equal (≠) - teng emas belgisi, tibbiy testlarda yoki tahlil natijalarida ishlatiladi',
    r'≡': 'Identical To (≡) - tenglikni yoki moslikni ifodalashda ishlatiladi',
    r'≤': 'Less Than or Equal To (≤) - kichik yoki teng belgisi, o\'lchovlar yoki parametrlar uchun',
    r'≥': 'Greater Than or Equal To (≥) - katta yoki teng belgisi, o\'lchovlar yoki parametrlar uchun',
    r'♀': 'Female sign (♀) - ayollarni ifodalash uchun ishlatiladi (masalan, ginekologik muolajalar)',
    r'♂': 'Male sign (♂) - erkaklarni ifodalash uchun ishlatiladi (masalan, urologik muolajalar)',
    r'⇨': 'Rightwards Arrow (⇨) - yo\'l yoki jarayonning davom etishini ko\'rsatish uchun ishlatiladi',
    r'⇦': 'Leftwards Arrow (⇦) - qarama-qarshi yo\'lni ko\'rsatish uchun',
    r'→': 'Rightwards Arrow (→) - jarayonning natijasini yoki yo\'lni ko\'rsatish uchun',
    r'←': 'Leftwards Arrow (←) - yo\'lni yoki o\'zgarishni ko\'rsatish uchun',
    r'↓': 'Down Arrow (↓) - pasayishni yoki kamayishni ko\'rsatish uchun',
    r'↑': 'Up Arrow (↑) - o\'sishni yoki ko\'tarilishni ko\'rsatish uchun',
    r'↔': 'Left Right Arrow (↔) - ikkita yo\'lni yoki qarama-qarshi yo\'nalishni ifodalash',
    r'↻': 'Clockwise Arrow (↻) - tibbiy holatlar yoki davolanish jarayonining aylanishi yoki qaytishi',
    r'↺': 'Counterclockwise Arrow (↺) - to\'g\'ri emas yoki qarama-qarshi yo\'nalishni ko\'rsatish uchun',
    r'π': 'Pi (π) - tibbiyotda geometrik hisob-kitoblar',
    r'∆': 'Delta (Δ) - o\'zgarishni ko\'rsatish uchun ishlatiladi (masalan, o\'zgaruvchan dori miqdori)',
    r'β': 'Beta (β) - beta variantlari, tibbiy tadqiqotlarda ko\'p ishlatiladi (masalan, beta blokatorlar)',
    r'α': 'Alpha (α) - alfa darajasi, ko\'pincha laboratoriya testlarida ishlatiladi',
    r'χ': 'Chi (χ) - tibbiyotda statistik tahlil yoki mutlaq natijalarni ifodalashda',
    r'ω': 'Omega (ω) - omega 3 yog\' kislotasi, antioksidantlar yoki boshqa fiziologik qiymatlar',
    r'∑': 'Summation (∑) - yig\'ish yoki qo\'shish uchun, masalan, statistik yig\'indi',
    r'λ': 'Lambda (λ) - tibbiyotda qatorlarni ifodalashda ishlatiladi, masalan, receptorlar',
    r'μ': 'Mu (μ) - mikro, o\'lchamlarni ifodalashda ishlatiladi (mikrogramm yoki mikrolitr)',
    r'Φ': 'Phi (Φ) - tibbiy hisoblash yoki miqdorlar uchun ishlatiladi (masalan, fluidlar)',
    r'θ': 'Theta (θ) - tibbiy parametrlar, o\'zgaruvchilar yoki normal holatlar',
    r'ω': 'Omega (ω) - energiya yoki organik moddalarning tibbiy tahlilida ishlatiladi',
    r'σ': 'Sigma (σ) - statistik tahlil va tibbiy testlar natijalarida ishlatiladi',
    r'ψ': 'Psi (ψ) - ruhiy holatlarni o\'lchashda ishlatiladi',
    r'μg': 'Microgram (μg) - kichik miqdorlar, masalan, dori dozasi',
    r'mg': 'Milligram (mg) - o\'lchov birligi, dorilarning dozasi uchun',
    r'kg': 'Kilogram (kg) - tana og\'irligini o\'lchash uchun ishlatiladi',
    r'cm': 'Centimeter (cm) - uzunlik yoki balandlikni o\'lchash uchun ishlatiladi',
    r'℃': 'Celsius (℃) - harorat o\'lchov birligi, tana harorati',
    r'℉': 'Fahrenheit (℉) - harorat o\'lchov birligi',
    r'π': 'Pi (π) - tibbiyotda geometrik hisob-kitoblar',
}

# Foydalanuvchidan tibbiy belgilarni qabul qilish va qaytarish
async def check_medical_symbols(update: Update, context: CallbackContext):
    text = update.message.text
    found_symbols = []
    for symbol, description in special_chars_dict.items():
        if re.search(symbol, text):
            found_symbols.append(description)

    # Topilgan belgilarni yuborish
    response = f"Topilgan maxsus belgilar va ularning tavsifi:\n" + "\n".join(found_symbols)
    response += f"\n\nTopilgan maxsus belgilar soni: {len(found_symbols)}"

    await update.message.reply_text(response)

# /start komandasi
async def start(update: Update, context: CallbackContext):
    await update.message.reply_text("Salom! Tibbiy belgilarni tekshirish uchun matn kiriting.")

# Asosiy funksiya
async def main():
    application = Application.builder().token("7729036153:AAHMxNmggw1kWiaDOwTc8j71PoHc6Eyy23s").build()

    application.add_handler(CommandHandler("start", start))
    application.add_handler(MessageHandler(filters.TEXT, check_medical_symbols))

    await application.run_polling()

if __name__ == "__main__":
    import nest_asyncio
    nest_asyncio.apply()  # Asinxron event loopni qayta ishlash uchun qo'llaniladi
    import asyncio
    asyncio.get_event_loop().run_until_complete(main())
