# MS VPS — Railway و Docker

## الملفات

- `vps_panel.py`: تطبيق Flask الرئيسي.
- `requirements.txt`: الاعتماديات المطلوبة.
- `Procfile` و`railway.json`: إعدادات البناء والتشغيل على Railway.
- `Dockerfile`: صورة Python جاهزة للتشغيل بالحاويات.
- `docker-compose.yml`: تشغيل محلي مع Volume دائم لبيانات اللوحة.
- `.dockerignore`: يستبعد البيانات المؤقتة والملفات السرية من صورة Docker.

## التشغيل باستخدام Docker Compose

```bash
docker compose up -d --build
```

بعدها افتح:

```text
http://localhost:20276
```

لإيقاف الخدمة:

```bash
docker compose down
```

يتم حفظ بيانات اللوحة في Volume باسم `panel_data`، لذلك لا تضيع عند إعادة إنشاء الحاوية. البوت معطّل افتراضيًا (`RUN_BOT=0`). إذا كنت تحتاجه، عيّن `RUN_BOT=1` وأضف `TELEGRAM_BOT_TOKEN` كمتغير بيئة دون وضعه داخل Git أو الصورة.

## التشغيل باستخدام Docker مباشرة

```bash
docker build -t ms-vps-panel .
docker run -d --name ms-vps-panel \
  -p 20276:20276 \
  -e RUN_BOT=0 \
  -v ms-vps-panel-data:/app/panel_data \
  ms-vps-panel
```

يمكن تغيير المنفذ الخارجي، مع إبقاء `PORT` مساويًا للمنفذ الداخلي أو تمريره صراحةً:

```bash
docker run -d --name ms-vps-panel \
  -p 8080:20276 \
  -e PORT=20276 \
  -v ms-vps-panel-data:/app/panel_data \
  ms-vps-panel
```

## النشر على Railway

1. ارفع **محتويات المجلد** إلى المستودع، وليس ملف ZIP داخل المستودع. يجب أن يكون `Dockerfile` في جذر المستودع وبنفس الأحرف الكبيرة والصغيرة.
2. اترك `railway.json` في الجذر؛ فهو يحدد `DOCKERFILE` و`Dockerfile` صراحةً.
3. في Railway اختر المستودع ثم نفّذ Redeploy. يجب أن يظهر في سجل البناء ما يشبه `Using detected Dockerfile!`.
4. أضف المتغيرات عند الحاجة:
   - `RUN_BOT=0` للتشغيل بدون بوت تيليجرام.
   - `TELEGRAM_BOT_TOKEN` إذا كان لديك بوت وتريد ربطه من خلال اللوحة.
   - `BASE_PATH=/app/panel_data` اختياري؛ تخزين Railway المحلي مؤقت ما لم تستخدم Volume أو قاعدة بيانات.
5. Railway يضبط `PORT` تلقائيًا، والتطبيق يستمع على `0.0.0.0`.

## ملاحظات مهمة

- لا تضع Bot Token أو كلمات المرور داخل GitHub أو داخل ملفات عامة.
- لا يوجد `bot.py` في الحزمة الحالية؛ لذلك لا يتم تشغيل البوت تلقائيًا.
- بيانات المستخدمين وملفات اللوحة تُنشأ داخل `panel_data/`، وقد تضيع عند إعادة النشر على Railway بدون Volume.
