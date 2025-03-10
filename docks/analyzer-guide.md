
```markdown
# راهنمای ماژول تحلیل پروژه رصد

این مستند توضیحات و راهنمای استفاده از ماژول تحلیل (Analyzer) پروژه رصد را ارائه می‌دهد.

## مرور کلی

ماژول تحلیل (Analyzer) مسئول پردازش و تحلیل عمیق توییت‌ها با استفاده از هوش مصنوعی و الگوریتم‌های آماری است. این ماژول شامل چهار بخش اصلی است:

1. **کلاینت Claude (ClaudeClient)**: ارتباط با Anthropic Claude API برای تحلیل عمیق متون
2. **مدیریت هزینه (CostManager)**: مدیریت بهینه هزینه API‌ها و انتخاب مدل مناسب
3. **تشخیص موج (WaveDetector)**: شناسایی موج‌های توییتری با الگوریتم‌های آماری
4. **تحلیلگر اصلی (TweetAnalyzer)**: مدیریت فرایند تحلیل و ارتباط با سایر ماژول‌ها

## فایل‌های اصلی

### `services/analyzer/claude_client.py`

کلاینت ارتباط با Claude API برای تحلیل عمیق متون:

- **تحلیل احساسات**: تشخیص دقیق احساسات متون فارسی و انگلیسی
- **استخراج موضوعات**: شناسایی موضوعات اصلی و کلیدواژه‌های متون
- **تحلیل دسته‌ای**: بهینه‌سازی هزینه با تحلیل چندین متن در یک درخواست
- **تحلیل موج**: تحلیل عمیق موج‌های توییتری با Extended Thinking

نمونه استفاده:

```python
from app.services.analyzer.claude_client import ClaudeClient

# ایجاد نمونه کلاینت
claude_client = ClaudeClient()

# تحلیل احساسات متن
sentiment_result = await claude_client.analyze_sentiment(
    "این محصول فوق‌العاده است و من از خرید آن بسیار راضی هستم", 
    language="fa"
)

# استخراج موضوعات
topics_result = await claude_client.extract_topics(
    "بحران آب در ایران به یک چالش جدی تبدیل شده است. کمبود بارش و خشکسالی در سال‌های اخیر موجب افت شدید منابع آبی شده است.",
    language="fa"
)

# تحلیل دسته‌ای
batch_results = await claude_client.analyze_batch(
    ["متن اول", "متن دوم", "متن سوم"],
    analysis_type="sentiment"
)
```

### `services/analyzer/cost_manager.py`

مدیریت هزینه API‌ها و انتخاب بهینه مدل:

- **ردیابی هزینه**: ثبت و ردیابی هزینه استفاده از API‌ها
- **مدیریت بودجه**: کنترل استفاده براساس بودجه تعیین شده
- **انتخاب مدل بهینه**: انتخاب هوشمند مدل Claude مناسب برای هر تحلیل
- **گزارش‌دهی**: تولید گزارش‌های مصرف و هزینه

نمونه استفاده:

```python
from app.services.analyzer.cost_manager import CostManager, ApiType, AnalysisType
from app.db.session import get_db

# ایجاد نمونه مدیریت هزینه
db_session = await get_db()
cost_manager = CostManager(db_session, daily_budget=10.0)  # بودجه روزانه 10 دلار
await cost_manager.initialize()

# بررسی وضعیت بودجه
budget_status = await cost_manager.check_budget()
print(f"Used: ${budget_status['total_usage']:.2f}, Remaining: ${budget_status['remaining']:.2f}")

# انتخاب مدل بهینه برای تحلیل
model, estimation = cost_manager.select_optimal_model(
    AnalysisType.SENTIMENT,
    text_length=1000,
    is_important=True
)
print(f"Selected model: {model}, Estimated cost: ${estimation['estimated_cost']:.4f}")

# ثبت استفاده از API
await cost_manager.record_usage(
    api_type=ApiType.CLAUDE,
    operation="analyze_sentiment",
    tokens_in=500,
    tokens_out=200,
    cost=0.015
)
```

### `services/analyzer/wave_detector.py`

تشخیص موج‌های توییتری و ایجاد هشدار:

- **تشخیص موج حجمی**: شناسایی افزایش ناگهانی حجم توییت‌ها
- **تشخیص موج احساسی**: شناسایی تغییرات قابل توجه در احساسات
- **ایجاد هشدار**: تولید هشدار برای موج‌های مهم
- **امتیازدهی**: امتیازدهی به موج‌ها براساس اهمیت آن‌ها

نمونه استفاده:

```python
from app.services.analyzer.wave_detector import WaveDetector
from app.db.session import get_db

# ایجاد نمونه تشخیص موج
db_session = await get_db()
wave_detector = WaveDetector(db_session)

# تشخیص موج‌های حجمی
volume_waves = await wave_detector.detect_volume_waves(
    keywords=["انتخابات", "دولت"],
    hours_back=12
)

# تشخیص موج‌های احساسی
sentiment_waves = await wave_detector.detect_sentiment_waves(
    keywords=["اقتصاد", "تورم"],
    hours_back=24
)

# اجرای تشخیص و ایجاد هشدار
alerts = await wave_detector.run_detection_and_create_alerts(
    keywords=["محیط زیست", "آلودگی هوا"],
    hours_back=6,
    min_importance=4.0
)
```

### `services/analyzer/analyzer.py`

سرویس اصلی تحلیل توییت‌ها:

- **تحلیل توییت**: تحلیل عمیق یک توییت خاص
- **تحلیل دسته‌ای**: تحلیل چندین توییت به صورت همزمان
- **پردازش صف**: دریافت و پردازش توییت‌ها از صف تحلیل
- **تشخیص موج و هشدار**: شناسایی موج‌های توییتری و ایجاد هشدار
- **تولید گزارش**: تولید گزارش‌های تحلیلی از داده‌ها

نمونه استفاده:

```python
from app.services.analyzer.analyzer import TweetAnalyzer
from app.services.analyzer.claude_client import ClaudeClient
from app.services.analyzer.cost_manager import CostManager
from app.services.analyzer.wave_detector import WaveDetector
from app.db.session import get_db

# ایجاد نمونه‌های مورد نیاز
db_session = await get_db()
claude_client = ClaudeClient()
cost_manager = CostManager(db_session)
wave_detector = WaveDetector(db_session)

# ایجاد نمونه تحلیلگر
analyzer = TweetAnalyzer(
    db_session=db_session,
    claude_client=claude_client,
    cost_manager=cost_manager,
    wave_detector=wave_detector,
    batch_size=50
)
await analyzer.initialize()

# تحلیل یک توییت
result = await analyzer.analyze_tweet(tweet_id=123)

# تحلیل دسته‌ای
results = await analyzer.batch_analyze_tweets([101, 102, 103, 104])

# پردازش صف تحلیل
processed_count = await analyzer.process_analysis_queue()

# تولید گزارش
report = await analyzer.generate_report(
    keywords=["انتخابات", "مجلس"],
    hours_back=24,
    include_tweets=True
)

# تشخیص موج و ایجاد هشدار
alerts = await analyzer.detect_waves_and_alert()

# اجرای تحلیلگر به صورت مداوم (در یک تسک جداگانه)
import asyncio
asyncio.create_task(analyzer.run_analyzer(sleep_time=60))
```

## جریان کار

فرایند تحلیل توییت‌ها به صورت زیر انجام می‌شود:

1. توییت‌های پردازش شده توسط ماژول پردازش به صف تحلیل افزوده می‌شوند
2. تحلیلگر به صورت مداوم توییت‌ها را از صف دریافت می‌کند
3. برای هر دسته توییت:
   - احساسات متن تحلیل می‌شود
   - موضوعات اصلی استخراج می‌شوند
   - نتایج در دیتابیس ذخیره می‌شوند
4. به صورت دوره‌ای، موج‌های توییتری تشخیص داده شده و هشدار ایجاد می‌شود
5. گزارش‌های تحلیلی براساس درخواست‌ها تولید می‌شوند

![نمودار جریان کار تحلیل](analyzer-flow.png)

## مدیریت هزینه و استفاده بهینه از API

ماژول تحلیل برای بهینه‌سازی هزینه‌ها و استفاده کارآمد از API، از رویکردهای زیر استفاده می‌کند:

1. **سیستم لایه‌بندی**: توییت‌ها براساس اهمیت دسته‌بندی می‌شوند و تحلیل‌های عمیق‌تر فقط روی توییت‌های مهم‌تر انجام می‌شود

2. **انتخاب هوشمند مدل**: برای هر تحلیل، مدل مناسب Claude براساس پارامترهای زیر انتخاب می‌شود:
   - طول متن
   - اهمیت تحلیل
   - وضعیت بودجه روزانه
   - نوع تحلیل

3. **تحلیل دسته‌ای**: به جای تحلیل تک‌تک توییت‌ها، آن‌ها در دسته‌های بزرگتر تحلیل می‌شوند

4. **پردازش صفی**: توییت‌ها ابتدا پیش‌پردازش می‌شوند و فقط توییت‌های مرتبط و غیراسپم به مرحله تحلیل می‌رسند

5. **کنترل بودجه**: مصرف API به صورت خودکار کنترل می‌شود و در صورت رسیدن به آستانه بودجه، استراتژی مصرف تغییر می‌کند

## تحلیل موج‌های توییتری

تشخیص و تحلیل موج‌های توییتری یکی از قابلیت‌های کلیدی این ماژول است:

1. **انواع موج‌ها**:
   - **موج حجمی**: افزایش ناگهانی تعداد توییت‌ها
   - **موج احساسی**: تغییر قابل توجه در احساسات توییت‌ها

2. **پارامترهای تشخیص**:
   - **آستانه حجمی**: ضریب افزایش تعداد توییت‌ها (پیش‌فرض: 2.0)
   - **آستانه احساسی**: میزان تغییر در احساسات (پیش‌فرض: 0.3)
   - **حداقل توییت**: تعداد حداقل توییت برای تشخیص موج (پیش‌فرض: 10)
   - **پنجره زمانی**: بازه زمانی بررسی به دقیقه (پیش‌فرض: 60)

3. **تحلیل عمیق موج**:
   - بررسی کاربران تأثیرگذار در موج
   - شناسایی موضوعات اصلی مرتبط با موج
   - تحلیل هوشمند برای تشخیص طبیعی یا هماهنگ بودن موج
   - پیش‌بینی روند آینده موج

## راه‌اندازی و اجرای تحلیلگر

برای راه‌اندازی و اجرای تحلیلگر به صورت مداوم، از کد زیر استفاده کنید:

```python
import asyncio
from app.db.session import get_db
from app.services.analyzer.claude_client import ClaudeClient
from app.services.analyzer.cost_manager import CostManager
from app.services.analyzer.wave_detector import WaveDetector
from app.services.analyzer.analyzer import TweetAnalyzer

async def run_analyzer():
    # ایجاد اتصال‌ها
    db_session = await get_db()
    claude_client = ClaudeClient()
    cost_manager = CostManager(db_session)
    await cost_manager.initialize()
    wave_detector = WaveDetector(db_session)
    
    # ایجاد تحلیلگر
    analyzer = TweetAnalyzer(
        db_session=db_session,
        claude_client=claude_client,
        cost_manager=cost_manager,
        wave_detector=wave_detector,
        batch_size=50
    )
    await analyzer.initialize()
    
    try:
        # اجرای تحلیلگر
        await analyzer.run_analyzer(sleep_time=60)
    finally:
        # بستن اتصال‌ها
        await analyzer.close()

# اجرا در event loop
asyncio.run(run_analyzer())
```

## نکات تکمیلی

1. **پشتیبانی از زبان فارسی**: تمام تحلیل‌ها به صورت بهینه برای زبان فارسی پیاده‌سازی شده‌اند.

2. **Extended Thinking**: برای تحلیل‌های عمیق موج‌ها از قابلیت Extended Thinking مدل Claude استفاده می‌شود.

3. **مدیریت خطا**: سیستم مدیریت خطای قوی در تمام فرایندها پیاده‌سازی شده است.

4. **ثبت وقایع**: تمام رویدادها و خطاها با جزئیات ثبت می‌شوند.

5. **مقیاس‌پذیری**: سیستم برای مقیاس‌پذیری و پردازش حجم بالای توییت طراحی شده است.
```

### گام 2: به‌روزرسانی مستندات اصلی پروژه

فایل `docs/project-overview.md` را با محتوای زیر ایجاد کنید:

```markdown
# مستندات جامع پروژه رصد

## معرفی پروژه

پروژه "رصد" یک پلتفرم هوشمند برای پایش، تحلیل و گزارش‌دهی فعالیت‌های توییتر مرتبط با سازمان‌ها یا موضوعات خاص است. این سیستم به سازمان‌ها کمک می‌کند تا:
- موج‌های انتقادی و فعالیت‌های مشکوک را به سرعت شناسایی کنند
- افکار عمومی درباره موضوعات مختلف را بهتر درک کنند
- روندها و الگوهای کاربران را تحلیل نمایند
- براساس داده‌های دقیق، استراتژی‌های ارتباطی مناسب را اتخاذ کنند

## معماری پروژه

سیستم از پنج ماژول اصلی تشکیل شده است که به صورت مستقل اما مرتبط با یکدیگر کار می‌کنند:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  جمع‌آوری        │     │  پردازش         │     │  تحلیل          │
│  TwitterAPI.io  │────▶│  فیلترسازی      │────▶│  Claude API     │
│  وب‌هوک/وبسوکت   │     │  اولویت‌بندی     │     │  سیستم چندلایه‌ای │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        ▲                        ▲                      │
        │                        │                      │
        └────────────────────────┼──────────────────────┘
                                 │                
                                 ▼
┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │
│  فرانت‌اند       │◀────│  بک‌اند          │
│  داشبورد        │     │  REST API       │
│  مدیریت تنظیمات │     │  مدیریت کاربران │
└─────────────────┘     └─────────────────┘
```

### ماژول‌های اصلی

1. **ماژول جمع‌آوری (Collector)**
   - جمع‌آوری توییت‌ها با استفاده از TwitterAPI.io
   - پایش کلیدواژه‌ها و کاربران
   - ذخیره‌سازی داده‌ها در دیتابیس

2. **ماژول پردازش (Processor)**
   - فیلترینگ اسپم و محتوای نامرتبط
   - تشخیص زبان و پیش‌پردازش متن
   - محاسبه امتیاز اهمیت توییت‌ها

3. **ماژول تحلیل (Analyzer)**
   - تحلیل احساسات و موضوعات با Claude API
   - تشخیص موج‌های توییتری با الگوریتم‌های آماری
   - مدیریت هزینه API با سیستم چندلایه‌ای

4. **ماژول بک‌اند (Backend)**
   - API‌های RESTful برای دسترسی به قابلیت‌های سیستم
   - مدیریت کاربران و احراز هویت
   - مدیریت هشدارها و گزارش‌ها

5. **ماژول فرانت‌اند (Frontend)**
   - داشبورد مدیریتی
   - نمودارهای تحلیلی
   - مدیریت تنظیمات و کلیدواژه‌ها

### فناوری‌های اصلی
- **TwitterAPI.io**: برای جمع‌آوری داده‌های توییتر
- **Claude API (Anthropic)**: برای تحلیل‌های عمیق و هوشمند
- **FastAPI**: برای ایجاد API‌های بک‌اند
- **SQLAlchemy**: برای ارتباط با دیتابیس
- **Redis**: برای کش، صف، و پیام‌رسانی بین سرویس‌ها
- **React**: برای توسعه فرانت‌اند
- **Docker**: برای کانتینرایز کردن سرویس‌ها

## وضعیت فعلی پروژه

### پیشرفت ماژول‌ها
- **زیرساخت پایه**: 100% تکمیل شده
- **ماژول جمع‌آوری (Collector)**: 100% تکمیل شده
- **ماژول پردازش (Processor)**: 100% تکمیل شده
- **ماژول تحلیل (Analyzer)**: 100% تکمیل شده
- **ماژول بک‌اند (Backend)**: 30% تکمیل شده
- **ماژول فرانت‌اند (Frontend)**: 0% تکمیل شده

### ویژگی‌های تکمیل شده

- ✅ **جمع‌آوری توییت براساس کلیدواژه**: پیاده‌سازی کامل جستجو و ذخیره‌سازی توییت‌ها
- ✅ **جمع‌آوری اطلاعات کاربران**: دریافت و ذخیره پروفایل‌های کاربران مرتبط
- ✅ **پردازش محتوای توییت**: استخراج متادیتا و ذخیره‌سازی ساختاریافته
- ✅ **تشخیص اسپم و محتوای نامرتبط**: فیلترینگ پایه برای بهبود کیفیت داده‌ها
- ✅ **تحلیل مرتبط بودن با کلیدواژه‌ها**: محاسبه امتیاز ارتباط
- ✅ **محاسبه امتیاز اهمیت توییت‌ها**: الگوریتم اولویت‌بندی توییت‌ها
- ✅ **تحلیل احساسات**: تشخیص احساسات مثبت/منفی/خنثی توییت‌ها با هوش مصنوعی
- ✅ **تشخیص موج‌های حجمی**: شناسایی افزایش ناگهانی حجم توییت‌ها
- ✅ **شناسایی موضوعات اصلی**: استخراج و دسته‌بندی موضوعات
- ✅ **سیستم هشدار**: ایجاد هشدار برای موج‌های مهم و تغییرات قابل توجه
- ✅ **مدیریت هزینه API**: سیستم بهینه‌سازی مصرف API با انتخاب هوشمند مدل‌ها

### ویژگی‌های در حال توسعه
- ⏳ **API‌های بک‌اند**: اندپوینت‌های دسترسی به تحلیل‌ها، توییت‌ها و هشدارها
- ⏳ **مدیریت کاربران سیستم**: احراز هویت و کنترل دسترسی
- ⏳ **داشبورد مدیریتی**: رابط کاربری برای مدیریت و مشاهده تحلیل‌ها
- ⏳ **مدیریت تنظیمات**: رابط‌های مدیریت کلیدواژه‌ها و تنظیمات سیستم

## قابلیت‌های کلیدی

### تحلیل احساسات با هوش مصنوعی
سیستم با استفاده از API کلود تحلیل دقیقی از احساسات توییت‌ها ارائه می‌دهد:
- تشخیص احساسات مثبت، منفی، خنثی و ترکیبی
- محاسبه امتیاز احساسات بین -1 تا 1
- تشخیص میزان اطمینان
- توضیحات متنی برای دلیل تشخیص احساس

### تشخیص موج‌های توییتری
سیستم قادر به شناسایی انواع موج‌های توییتری است:
- **موج‌های حجمی**: افزایش ناگهانی تعداد توییت‌ها درباره یک موضوع
- **موج‌های احساسی**: تغییر قابل توجه در احساسات توییت‌ها
- **موج‌های هماهنگ**: توییت‌های هماهنگ شده که می‌تواند نشانه کمپین‌های سازماندهی شده باشد

### سیستم هشدار هوشمند
براساس تشخیص موج‌ها، سیستم به صورت خودکار هشدارهایی ایجاد می‌کند:
- درجه‌بندی هشدارها (کم، متوسط، زیاد)
- توضیحات کامل درباره دلیل هشدار
- پیشنهادات اقدامات بعدی
- ارتباط با توییت‌های مرتبط

### مدیریت هزینه API
سیستم مدیریت هزینه هوشمند API:
- استفاده از سیستم چندلایه‌ای و انتخاب مدل مناسب براساس نوع تحلیل
- کنترل بودجه روزانه و تخصیص هوشمند منابع
- تحلیل دسته‌ای برای کاهش هزینه
- تمرکز بر توییت‌های مهم‌تر برای تحلیل‌های عمیق‌تر

## راهنمای راه‌اندازی

### پیش‌نیازها
- Python 3.9+
- PostgreSQL
- Redis
- API key برای TwitterAPI.io
- API key برای Anthropic Claude

### تنظیم محیط توسعه
1. کلون کردن مخزن گیت:
   ```bash
   git clone https://github.com/your-org/rasad.git
   cd rasad
   ```

2. ایجاد محیط مجازی پایتون:
   ```bash
   python -m venv venv
   source venv/bin/activate  # در لینوکس
   venv\Scripts\activate  # در ویندوز
   ```

3. نصب وابستگی‌ها:
   ```bash
   pip install -r requirements.txt
   ```

4. ایجاد فایل `.env`:
   ```
   DEBUG=True
   SECRET_KEY=your_secret_key
   POSTGRES_SERVER=localhost
   POSTGRES_USER=postgres
   POSTGRES_PASSWORD=your_password
   POSTGRES_DB=rasad
   REDIS_HOST=localhost
   REDIS_PORT=6379
   TWITTER_API_KEY=your_twitter_api_key
   CLAUDE_API_KEY=your_claude_api_key
   DAILY_BUDGET=10.0
   ```

5. ایجاد دیتابیس:
   ```bash
   createdb rasad  # در PostgreSQL
   ```

### اجرای سرویس‌ها

**اجرای بک‌اند**:
```bash
cd app
python main.py
```

**اجرای جمع‌آوری کننده**:
```bash
cd scripts
python run_collector.py
```

**اجرای پردازشگر**:
```bash
cd scripts
python run_processor.py
```

**اجرای تحلیلگر**:
```bash
cd scripts
python run_analyzer.py
```

## مستندات تکمیلی

- [راهنمای ماژول زیرساخت](basic-infrastructure-guide.md)
- [راهنمای ماژول جمع‌آوری](collector-guide.md)
- [راهنمای ماژول پردازش](processor-guide.md)
- [راهنمای ماژول تحلیل](analyzer-guide.md)
- [راهنمای ماژول بک‌اند](backend-guide.md)
```

## بخش 4: مرحله بعدی توسعه

حال که ماژول تحلیل را کامل کرده‌ایم، نوبت به تکمیل ماژول بک‌اند می‌رسد. در این بخش، مراحل بعدی توسعه را توضیح می‌دهیم.

### گام بعدی: تکمیل ماژول بک‌اند

مرحله بعدی توسعه، تمرکز بر ماژول بک‌اند است. در این مرحله، شما باید اقدامات زیر را انجام دهید:

1. **تکمیل اندپوینت‌های API برای دسترسی به قابلیت‌های سیستم**:
   - اندپوینت‌های توییت‌ها برای جستجو و فیلتر کردن
   - اندپوینت‌های تحلیل‌ها برای دریافت نتایج تحلیل و درخواست تحلیل‌های جدید
   - اندپوینت‌های هشدارها و موج‌ها برای دریافت و مدیریت هشدارها
   - اندپوینت‌های تنظیمات برای مدیریت کلیدواژه‌ها و پارامترهای سیستم

2. **پیاده‌سازی احراز هویت و مدیریت کاربران**:
   - سیستم احراز هویت JWT
   - سطوح دسترسی و مجوزها
   - مدیریت کاربران سیستم

3. **ایجاد سرویس‌های میانی برای ارتباط بین اندپوینت‌ها و ماژول‌های اصلی**:
   - سرویس‌های تبدیل و ارائه داده
   - مدیریت داده و ارتباط با دیتابیس

4. **پیاده‌سازی تست‌های واحد و انسجام**:
   - تست اندپوینت‌های API
   - تست سناریوهای مختلف

5. **مستندسازی API با Swagger و ReDoc**:
   - مستندات کامل اندپوینت‌ها
   - نمونه‌های درخواست و پاسخ

برای شروع، باید فایل‌های زیر را در مسیر `app/api/v1/` ایجاد یا تکمیل کنید:

1. `app/api/v1/auth.py`: اندپوینت‌های احراز هویت و مدیریت کاربران
2. `app/api/v1/tweets.py`: اندپوینت‌های جستجو و مدیریت توییت‌ها
3. `app/api/v1/analysis.py`: اندپوینت‌های تحلیل و گزارش‌گیری
4. `app/api/v1/waves.py`: اندپوینت‌های موج‌ها و هشدارها
5. `app/api/v1/settings.py`: اندپوینت‌های مدیریت تنظیمات سیستم

پس از تکمیل ماژول بک‌اند، می‌توانید به سراغ ماژول فرانت‌اند بروید.

## نکات پایانی

تبریک! شما موفق شدید ماژول تحلیل پروژه رصد را کامل کنید. این ماژول قلب هوشمند سیستم است که قابلیت‌های تحلیل عمیق توییت‌ها و تشخیص موج‌ها را فراهم می‌کند.

برای ادامه توسعه، روش‌های زیر را به کار ببرید:

1. **توسعه مرحله‌ای**: هر ماژول را به طور کامل تکمیل کنید قبل از شروع ماژول بعدی
   
2. **تست‌نویسی همزمان**: برای هر بخش جدید، تست‌های مربوطه را نیز بنویسید

3. **مستندسازی همگام**: مستندات را همگام با توسعه به‌روزرسانی کنید

4. **یکپارچه‌سازی مداوم**: اطمینان حاصل کنید که همه بخش‌ها با یکدیگر سازگار هستند

مستندات کامل و ساختار مناسب پروژه به شما کمک می‌کند توسعه را سریع‌تر و با کیفیت بالاتر انجام دهید.