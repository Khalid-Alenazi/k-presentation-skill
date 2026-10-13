# K Presentation Skill

A Claude skill that turns any source material (deck, PDF, notes, screenshots) into a **self-contained, video-style HTML film**: animated SVG scenes, synced captions, ElevenLabs v4 narration, optional background music, and Elm brand identity.

سكل لـ Claude يحوّل أي محتوى (عرض، PDF، ملاحظات، صور) إلى **فيلم HTML يشتغل كفيديو**: مشاهد SVG متحركة، ترجمة متزامنة، تعليق صوتي بـ ElevenLabs v4، موسيقى خلفية اختيارية، وهوية علم.

## What it does / وش يسوي
- Asks first: **language**, **duration**, **background music** / يسأل أول شي: اللغة، المدة، الموسيقى
- Storyboard for approval before building / يعرض الـ storyboard قبل البناء
- Video player: Play bottom-left, Fullscreen bottom-right, CC captions toggle, Space / ← → (±5s) / F / C (captions) / M
- Personas: each solution speaks in first person with its own color, emblem and voice / كل حل يتكلم بشخصيته
- Suggests ElevenLabs voices + gives one full script with v4 audio tags / يقترح الأصوات ويعطيك سكربت كامل بالـ tags
- You generate the audio, send it back, and Claude merges and re-syncs everything / تولّد الصوت وترجعه، ويدمجه ويرتبه

## Install / التثبيت
**Claude.ai:** download this folder as a ZIP, then upload it in **Settings → Capabilities → Skills**.
حمّل المجلد ZIP وارفعه من **Settings ← Capabilities ← Skills**.

**Claude Code:** copy the folder to `~/.claude/skills/k-presentation-skill/`.

## Use / الاستخدام
> Make me a video HTML about [topic], source attached.
> سوّ لي فيديو HTML عن [الموضوع]، المصدر مرفق.

## Tips / نصائح
- Attach the logo, fonts and real screenshots. / ارفق الشعار والخطوط والصور الحقيقية.
- Say upfront what must not change (numbers, names). / حدد اللي ما يتغير من البداية.
