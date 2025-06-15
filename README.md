from telegram import Update, ReplyKeyboardMarkup, ReplyKeyboardRemove
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, ContextTypes, filters, ConversationHandler

# Các trạng thái cuộc trò chuyện
CHOICE, RO_BUX, ACC_ROBLOX, GAMEPASS, THANH_TOAN = range(5)

TOKEN = '8197684665:AAF2DOtfeC1b2aA_MGV8wxl62D1EQTJoF5s'  # Thay bằng token của bạn

# Menu sản phẩm
menu = [
    ['💎 Mua Robux', '🎮 Mua Acc Roblox'],
    ['🎁 Mua Gamepass']
]

# Bảng giá Robux
robux_gia = (
    "1️⃣ Mua Robux qua GROUP:\n"
    "100 Robux: 50k\n"
    "200 Robux: 90k\n"
    "500 Robux: 220k\n"
    "Lưu ý: Robux sẽ nhận trong vòng 24-48h\n\n"
    "2️⃣ Mua Robux nạp thẳng GAME:\n"
    "100 Robux: 60k\n"
    "200 Robux: 110k\n"
    "500 Robux: 260k\n"
    "Lưu ý: Robux vào tài khoản ngay sau khi thanh toán\n"
)

# Bảng giá ACC
acc_gia = (
    "Các gói ACC:\n"
    "🔸 100k-200k: Nick thường, random skin\n"
    "🔸 300k-500k: Nick VIP, full pet\n"
    "🔸 700k-1tr: Nick VIP + Robux sẵn, chỉ việc vào chơi\n"
)

# Bảng giá Gamepass
gamepass_gia = (
    "Gamepass hiện có:\n"
    "- Blox Fruits: 300k\n"
    "- Pet Simulator X: 400k\n"
    "- Adopt Me: 250k\n"
    "- Brookhaven: 200k\n"
)

# Hướng dẫn thanh toán
huongdan_tt = (
    "Vui lòng chuyển khoản theo cú pháp:\n"
    "[Telegram Username] + [Sản phẩm]\n"
    "Ngân hàng: ABC Bank\n"
    "STK: 123456789\n"
    "Chủ TK: NGUYEN VAN A\n"
    "Sau khi thanh toán, bạn nhấn vào: 'Tôi đã thanh toán'"
)

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "🎯 Chào mừng bạn đến với SHOP ROBLOX VIP!\n\nBạn cần mua gì hôm nay?",
        reply_markup=ReplyKeyboardMarkup(menu, one_time_keyboard=True, resize_keyboard=True)
    )
    return CHOICE

async def choice(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text

    if text == '💎 Mua Robux':
        await update.message.reply_text(robux_gia + "\n\n" + huongdan_tt, 
                                        reply_markup=ReplyKeyboardMarkup([['Tôi đã thanh toán']], resize_keyboard=True))
        return THANH_TOAN

    elif text == '🎮 Mua Acc Roblox':
        await update.message.reply_text(acc_gia + "\n\n" + huongdan_tt, 
                                        reply_markup=ReplyKeyboardMarkup([['Tôi đã thanh toán']], resize_keyboard=True))
        return THANH_TOAN

    elif text == '🎁 Mua Gamepass':
        await update.message.reply_text(gamepass_gia + "\n\n" + huongdan_tt, 
                                        reply_markup=ReplyKeyboardMarkup([['Tôi đã thanh toán']], resize_keyboard=True))
        return THANH_TOAN

    else:
        await update.message.reply_text("Vui lòng chọn đúng dịch vụ trong menu.")
        return CHOICE

async def thanh_toan(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("✅ Vui lòng đợi hệ thống kiểm tra thanh toán...")

    # ⚠ Đây là chỗ bạn cần lập trình check thanh toán tự động
    # Ở bản demo tạm thời ta kiểm tra thủ công
    # Giả lập kiểm tra chuyển khoản thành công:
    thanh_cong = True  # ở đây là check API ngân hàng

    if thanh_cong:
        await update.message.reply_text(
            "💎 Đã nhận thanh toán!\n\nĐây là sản phẩm của bạn:\n"
            "👉 Tài khoản: user123\n👉 Mật khẩu: pass123\n"
            "👉 Lưu ý: vui lòng đổi mật khẩu ngay khi đăng nhập."
        )
        await update.message.reply_text("🙏 Cảm ơn bạn đã ủng hộ shop. Hẹn gặp lại ở lần sau ❤️",
                                        reply_markup=ReplyKeyboardRemove())
        return ConversationHandler.END

    else:
        await update.message.reply_text("❌ Hệ thống chưa nhận được thanh toán. Vui lòng kiểm tra lại hoặc liên hệ admin.\n"
                                        "⚠ Nếu cố tình fake bill - shop sẽ báo cáo Telegram và từ chối phục vụ về sau.")
        return ConversationHandler.END

async def cancel(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text('Hẹn gặp lại bạn sau ❤️', reply_markup=ReplyKeyboardRemove())
    return ConversationHandler.END

if __name__ == '__main__':
    app = ApplicationBuilder().token(TOKEN).build()

    conv_handler = ConversationHandler(
        entry_points=[CommandHandler('start', start)],
        states={
            CHOICE: [MessageHandler(filters.TEXT & ~filters.COMMAND, choice)],
            THANH_TOAN: [MessageHandler(filters.TEXT & ~filters.COMMAND, thanh_toan)],
        },
        fallbacks=[CommandHandler('cancel', cancel)]
    )

    app.add_handler(conv_handler)

    print("BOT ĐANG CHẠY ...")
    app.run_polling()