# مصادر تنفيذ الوسيط — 6 نوفمبر 2026

حصة Workers Free تبلغ حاليًا 100 ألف طلب يوميًا و10ms وقت CPU؛ انتظار الشبكة لا يدخل في CPU. هذا سقف Worker وليس حصة Gemini:
https://developers.cloudflare.com/workers/platform/limits/

Secrets تحفظ القيم النصية المشفرة وتوفرها لكود Worker، دون تضمينها في الإضافة:
https://developers.cloudflare.com/workers/configuration/secrets/

Durable Objects ذات SQLite متاحة في Workers Free؛ عند تجاوز حد مجاني تتوقف العمليات، ولها حصص مستقلة للطلبات والمدة والتخزين:
https://developers.cloudflare.com/durable-objects/platform/pricing/
https://developers.cloudflare.com/durable-objects/get-started/

الحدود الصارمة بين المواقع تحتاج Durable Objects؛ Rate Limiting binding وحده تقريبي ومحلي للموقع الجغرافي:
https://developers.cloudflare.com/workers/runtime-apis/bindings/rate-limit/

حصص Gemini للمشروع وليس لكل مفتاح. راجع RPM/TPM/RPD الفعلية في حساب المالك:
https://ai.google.dev/gemini-api/docs/rate-limits

Gemini 3.8 Flash TTS وCharon وspeech_metadata مع Interactions API:
https://ai.google.dev/gemini-api/docs/speech-generation
https://ai.google.dev/api/interactions

مجانية النموذج الحالية وحدود سياسة البيانات:
https://ai.google.dev/gemini-api/docs/pricing
