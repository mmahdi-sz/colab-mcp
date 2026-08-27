# سرور Colab MCP (نسخه ارتقایافته و پایدار)

[![پایتون](https://img.shields.io/badge/پایتون-3.10%2B-blue.svg)](https://python.org)
[![پروتکل MCP](https://img.shields.io/badge/سازگار%20با-پروتکل%20MCP-green.svg)](https://modelcontextprotocol.io)
[![مجوز](https://img.shields.io/badge/مجوز-Apache%202.0-orange.svg)](LICENSE)
[![تست‌شده روی](https://img.shields.io/badge/تست‌شده%20روی-Google%20Antigravity%20%7C%20Claude%20Code-purple.svg)](#کلاینت‌های-پشتیبانی‌شده)

**[English](README.md)** | **[فارسی](README.fa.md)**

این پروژه یک سرور رسمی و اصلاح‌شده بر پایه استاندارد [Model Context Protocol (MCP)](https://modelcontextprotocol.io) است که امکان اتصال زنده و دوطرفه میان ایجنت‌های هوش مصنوعی محلی (مانند **Google Antigravity** و **Claude**) با نوتبوک‌های فعال در مرورگر [Google Colab](https://colab.research.google.com) را فراهم می‌کند.

---

## 🌟 اصلاحات و مزایای کلیدی در این نسخه

این نسخه به طور ویژه برای رفع چالش‌های محیط‌های لینوکسی، دسکتاپ‌های ریموت (KDE/GNOME/xrdp) و استانداردهای سخت‌گیرانه JSON-RPC بهینه‌سازی شده است:

1. **جلوگیری از مسمومیت استریم STDIO:**
   - حذف خودکار بنرهای رنگی متنی و هشدارهای فریم‌ورک FastMCP به منظور حفظ ساختار بی‌نقص پیام‌های JSON-RPC.
2. **یکپارچه‌سازی پورت‌های دوگانه IPv4 و IPv6:**
   - رفع باگ تخصیص پورت‌های ناهماهنگ در لینوکس (`127.0.0.1` در برابر `::1`) با تخصیص‌دهنده یکتای پورت (`_get_free_port`).
3. **سازگاری با محیط‌های بدون نمایشگر (Headless) و سرورهای ریموت:**
   - ایزوله‌سازی فراخوانی مرورگر گرافیکی و ذخیره لینک مستقیم اتصال در فایل `/tmp/colab_mcp_url.txt`.
4. **پایداری کامل هندشیک با گوگل کولب:**
   - نادیده گرفتن امن پیام‌های فریم غیررسمی فرانت‌اند کولب (مانند `server/discover`) بدون ریست شدن یا قطع ارتباط سرور.

---

## 🖥️ کلاینت‌های پشتیبانی‌شده

تمامی دستیارهایی که از استاندارد ابزارهای داینامیک (`notifications/tools/list_changed`) پشتیبانی می‌کنند:
* **Google Antigravity**
* **Claude Code & Claude Desktop**
* **Gemini CLI**
* **Cursor & Windsurf**
* **Goose / Zed**

---

## 🚀 راهنمای نصب و راه‌اندازی سریع

### ۱. پیش‌نیازها
* پایتون نسخه ۳.۱۰ یا بالاتر
* ابزار [`uv`](https://docs.astral.sh/uv/) (پیشنهادی) یا `pip`
* مرورگر گوگل کروم یا هر مرورگر مدرن

### ۲. نصب پکیج

#### روش اول: نصب مستقیم از سورس محلی یا گیت‌هاب (پیشنهادی)
```bash
uv venv ~/.local/share/colab-mcp-venv --python python3
uv pip install -e /path/to/colab-mcp --python ~/.local/share/colab-mcp-venv/bin/python
```

### ۳. ساخت اسکریپت رانر ایزوله (Wrapper)
برای اطمینان از سلامت استریم استاندارد JSON-RPC، اسکریپت زیر را اجرا کنید:

```bash
cat << 'WRAPPER_EOF' > ~/.local/share/colab-mcp-venv/bin/colab-mcp-wrapper
#!/bin/bash
export NO_COLOR=1
export PYTHONWARNINGS="ignore"
export FASTMCP_DISABLE_VERSION_CHECK=1
exec /home/$USER/.local/share/colab-mcp-venv/bin/colab-mcp "$@" 2>>/tmp/colab_mcp_stderr.log
WRAPPER_EOF

chmod +x ~/.local/share/colab-mcp-venv/bin/colab-mcp-wrapper
```

---

## ⚙️ تنظیمات در نرم‌افزارهای هوش مصنوعی

### تنظیم در Google Antigravity (`~/.gemini/config/mcp_config.json`)
```json
{
  "mcpServers": {
    "colab-mcp": {
      "command": "/home/YOUR_USER/.local/share/colab-mcp-venv/bin/colab-mcp-wrapper",
      "args": [],
      "env": {
        "FASTMCP_DISABLE_VERSION_CHECK": "1",
        "NO_COLOR": "1"
      },
      "timeout": 30000
    }
  }
}
```

### تنظیم در Claude Desktop (`claude_desktop_config.json`)
```json
{
  "mcpServers": {
    "colab-mcp": {
      "command": "/home/YOUR_USER/.local/share/colab-mcp-venv/bin/colab-mcp-wrapper",
      "args": []
    }
  }
}
```

---

## 📖 نحوه اتصال به نوتبوک کولب

### روش ۱: اتصال از طریق لینک (خودکار)
۱. در چت به هوش مصنوعی بگویید: *«به نوتبوک کولب من وصل شو»*.
۲. لینکی به شکل زیر باز می‌شود یا به شما ارائه می‌گردد:
   ```text
   https://colab.research.google.com/notebooks/empty.ipynb#mcpProxyToken=TOKEN&mcpProxyPort=PORT
   ```
۳. در پنجره بازشده روی دکمه **Connect** کلیک کنید.

### روش ۲: اتصال به تبِ از قبل بازشده در مرورگر
۱. در تب نوتبوک کولب، کلیدهای **`Ctrl + Shift + P`** را فشار دهید.
۲. عبارت **`Connect to a local Colab MCP server`** را جستجو و اینتر کنید.
۳. رشته توکن ترکیبی (`TOKEN&PORT`) داده‌شده توسط ایجنت را در کادر پیست کرده و دکمه **Connect** را بزنید.
۴. پیام سبز رنگ **`Connected to the local Colab MCP server`** ظاهر می‌شود و کنترل زنده نوتبوک فعال می‌گردد.

---

## 📄 مجوز (License)
این پروژه تحت مجوز [Apache License, Version 2.0](LICENSE) منتشر شده است.
