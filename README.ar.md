```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**وكيل ذكاء اصطناعي قابل للتهيئة لطرفيتك.**
احضر مفاتيح واجهة البرمجة الخاصة بك: Z.ai (Zhipu GLM)، وAnthropic، وOpenAI، وOpenRouter، وOllama محلية، أو أي خادم متوافق مع OpenAI. لينكس وTermux أولاً.

[English](README.md) · العربية · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **ما قبل-ألفا، الإصدار 0.1.0-alpha.1.** ما تم تنفيذه: واجهة TUI أصلية، ودردشة مع عدة مزوّدين، وحلقة وكيل واعية بالأذونات، وأدوات محلية وجلسات محلية قابلة للاستئناف. واجهات البرمجة والإعدادات قد تتغير بعد. هذا نموذج أولي، وليس تكافؤاً في الميزات مع OpenCode أو Hermes.

*Marelle* هو الاسم الفرنسي للعبة القفز على المربعات. كل مربع خطوة.

## التثبيت

**Termux على أندرويد.** حمّل الملف التنفيذي الجاهز من صفحة [الإصدارات](https://github.com/S-I-L-S/marelle/releases)، أو ابنه على الهاتف. دليل خطوة بخطوة: [docs/TERMUX.md](docs/TERMUX.md).

**لينكس وأي مكان يعمل فيه Rust.** يتطلب Rust 1.85 أو أحدث:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

أو استنسخ المستودع وشغّل `cargo build --release`. الملف التنفيذي هو `target/release/marelle`، نحو 4 مِيبايت.

## أول دردشة

احفظ المفتاح في ملف لا يقرأه سواك، وأخبر Marelle أين هو. لا يخزّن Marelle مفتاحاً أبداً في إعداداته.

```sh
mkdir -p ~/.config/marelle
( umask 077; read -rsp "API key: " key; echo; printf '%s' "$key" > ~/.config/marelle/zai.key )
export MARELLE_PROVIDERS__ZAI__API_KEY_FILE=~/.config/marelle/zai.key

marelle providers                                    # what exists, and whether the key was found
marelle config set chat.provider zai
marelle config set chat.model glm-5.2
marelle                                                # open the interactive terminal UI
marelle chat -m zai/MODEL "Explain hopscotch in one sentence"
```

استبدل `MODEL` باسم نموذج من قائمة مزوّدك. طرق استخدام أخرى:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

الجواب يذهب إلى المخرج القياسي. التفكيرات، وعدّ الرموز، والملاحظات والأخطاء تذهب إلى المخرج الخاطئ؛ لذا `marelle chat ... > answer.txt` يحفظ الجواب وحده.

## أدوات الوكيل

يرسل Marelle تعريفات منظمة للأدوات عبر واجهات متوافقة مع OpenAI وAnthropic. طقم الأدوات المدمج:

| الأداة | الغاية |
| --- | --- |
| `terminal` | تنفيذ أمر صَدَفة من مجلد الإقلاع |
| `read_file`, `write_file`, `edit_file` | قراءة وتعديل ملفات نصية UTF-8 |
| `list_directory`, `search_text` | تفقّد ملفات المشروع والبحث فيها |
| `fetch_url` | جلب صفحة HTTP(S) عامة كنص (يتطلب `curl`) |
| `list_skills`, `load_skill` | اكتشاف وتحميل تعليمات محلية `SKILL.md` |

الأمر `/tools` يعرض قائمة الأدوات، و`/skills` يسرد المهارات المتاحة. توجيهات المشروع تُحمَّل من `AGENTS.md` و`CLAUDE.md` و`SOUL.md` و`.cursorrules` وملفات `.marelle/` مختارة متى كانت داخل مسارات مسموح بها.

أدوات الملفات محصورة في `permissions.allowed_paths` (افتراضياً: مجلد المشروع الحالي) وتحترم `permissions.denied_paths`. القراءة مسموحة افتراضياً؛ تغيير الملفات وأوامر الصَدَفة والجلب من الشبكة تسأل قبل التنفيذ. إذن الصَدَفة ينطبق على أوامر على المضيف وليس حاضنة لنظام الملفات: أبقه على `ask` إلا إذا وثقت النموذج بالتنفيذ عن وعي.

يتوقف الوكيل بعد `agent.max_turns` جولة بأدوات. ردود الأدوات تعامل كمحتوى غير موثوق. الطلبات تستخدم أجوبة نموذج مكتملة، بحيث لا تُجمَّع وسائط الأدوات من شظايا ناقصة من التيار.

## الجلسات

تُحفظ المحادثات محلياً افتراضياً بما في ذلك تاريخ نداءات الأدوات وإعدادات النموذج والجلسة. على يونكس مجلد الجلسات بوضع `0700` وملف JSON بوضع `0600`. يُحفظ الملف في `$XDG_STATE_HOME/marelle/sessions.json`، أو `~/.local/state/marelle/sessions.json` حين لا يكون `XDG_STATE_HOME` مضبوطاً. مفاتيح واجهة البرمجة لا تُكتب في هذا الملف أبداً. عطّل التخزين بـ:

```sh
marelle config set agent.persist_sessions false
```

رموز الخروج: `0` نجاح، `1` خطأ مزوّد أو شبكة، `2` خطأ استخدام أو إعداد.

## المزوّدون

| id | البروتوكول | متغير المفتاح | |
| --- | --- | --- | --- |
| `zai` | متوافق مع OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM)، دولي |
| `zai_coding` | متوافق مع OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai، نقطة نهاية متوافقة مع Anthropic |
| `zhipu` | متوافق مع OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel، الصين |
| `zhipu_coding` | متوافق مع OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan، الصين |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | متوافق مع OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | متوافق مع OpenAI | `OPENROUTER_API_KEY` | نماذج كثيرة خلف مفتاح واحد |
| `ollama` | متوافق مع OpenAI | لا شيء | Ollama على هذه الآلة |

نقاط النهاية لـZ.ai وZhipu تأتي من توثيق أطراف ثالثة، لا من صفحات المزوّد نفسه. قارنها مع خطتك وتجاوز أي حقل إن اختلفت.

كل حقل في المزوّد قابل للتغيير، والمزوّد الجديد ليس سوى اسم مع عنوان:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

حقول المزوّد الأخرى: `kind` (`openai-compatible` أو `anthropic`)، و`api_key_file`، و`auth` (`bearer`، `x-api-key`، `api-key`، `none`)، و`max_tokens_field`، و`stream_usage`، و`extra_body`. إن كان المتغير المسماى في `api_key_env` مضبوطاً فهو الغالب؛ وإلا يُقرأ الملف.

## الإعدادات

تأتي الإعدادات في طبقات: قيم افتراضية مدمجة، ثم متغيرات البيئة، ثم الأعلام. ملفات الإعدادات تنسد بين الافتراضات والبيئة.

- `MARELLE_CHAT__MODEL=zai/MODEL` تضبط `chat.model`. الشرطة السفلية المزدوجة تقوم مقام نقطة.
- `marelle config show [prefix]` يسرد الإعدادات والطبقة التي جاء منها كل واحد.
- `marelle config explain <key>` يعرض كل طبقة ضبطت مفتاحاً، من أدنى أولوية إلى أعلاها.

| المفتاح | الافتراضي | المعنى |
| --- | --- | --- |
| `chat.provider`, `chat.model` | لا شيء | مع من نتكلم. يمكن لـ`chat.model` أن يكون `provider/model`. |
| `chat.system` | لا شيء | موجّه النظام |
| `chat.max_tokens`, `chat.temperature` | لا شيء | غير مضبوط — يُترك للمزوّد |
| `chat.stream` | `true` | تدفّق جواب الحكم الاختياري؛ أدوار الوكيل بالأدوات تستخدم حالياً أجوبة مكتملة، كي تبقى نداءات الأدوات سليمة |
| `chat.show_reasoning`, `chat.show_usage` | `false` | التفكيرات وعدّ الرموز على المخرج الخاطئ |
| `agent.max_turns` | `20` | أقصى جولات أدوات لكل طلب مستخدم |
| `agent.persist_sessions` | `true` | حفظ المحادثات المحلية بين مرات التشغيل |
| `permissions.shell` | `ask` | تنفيذ أوامر الصَدَفة: `allow`، `ask`، `deny` |
| `permissions.files_read` | `allow` | قراءة الملفات: `allow`، `ask`، `deny` |
| `permissions.files_write` | `ask` | تعديل الملفات: `allow`، `ask`، `deny` |
| `permissions.network` | `ask` | جلب الروابط: `allow`، `ask`، `deny` |
| `permissions.allowed_paths` | `["."]` | الجذور المسموحة لأدوات الملفات |
| `permissions.denied_paths` | لا شيء | مسارات ملفات مسدودة صراحة |
| `network.connect_timeout_secs` | `15` | الوقت المسموح للاتصال |
| `network.idle_timeout_secs` | `300` | الصمت المسموح في انتظار البيانات |
| `network.proxy` | `system` | `system`، أو `none`، أو عنوان وسيط |
| `network.ca_bundle` | لا شيء | ملف إضافي من هيئات الشهادات الموثوقة |
| `network.allow_insecure_http` | `false` | السماح لمفتاح عبر `http` المجرد إلى مضيف أجنبي |
| `update.check` | `true` | سؤال GitHub وGitLab عن إصدار أحدث عند بدء جلسة تفاعلية |
| `update.github_repo` | `S-I-L-S/marelle` | `owner/repo` على GitHub يسأله الفحص عن أحدث إصدار |
| `update.gitlab_repo` | لا شيء | `owner/repo` على GitLab للفحص ذاته. الفراغ يطفئ GitLab |
| `update.timeout_secs` | `3` | كم ثانية قد ينفق الفحص على مصدر واحد |

يحمل Marelle قائمته الخاصة بهيئات الشهادات الموثوقة، فيعمل حتى حيث لا يوجد لها نظام، كما في Termux. `SSL_CERT_FILE` يضيف مزيداً.

إذا وُجد إصدار أحدث تذكره الجلسة مرة واحدة — الإصدار، وما لديك، وأين يُؤخذ. يُسأل الإصدار أولاً؛ المستودع الذي يوسم التزاماته بالوسوم فقط يعطي أحدث وسم. أي فشل يبقى صامتاً.

## مفاتيحك تبقى لك

- الإعدادات تسمّي متغيراً أو ملفاً يضم المفتاح، ولا المفتاح ذاته أبداً. إعداد مثل `providers.x.api_key` يُرفض.
- المفتاح لا يطبعه `config show` ولا `config explain` ولا `providers`، ويُقتطع من كل نص خطأ قادم من خادم.
- يرسل إلى عنوان المزوّد فقط، في ترويسة واحدة، ولا يمر عبر `http` المجرد إلى آلة أخرى إلا إذا سمحت.
- لا تتبع إعادة التوجيهات، فلا يمكن أن يُرسل المفتاح إلى مكان آخر.

## اللغات

الواجهة تتحدث الإنجليزية والروسية والفرنسية والأوكرانية. اختر بـ`--lang <code>` أو `MARELLE_LANG`؛ وإلا فقد تقرر `LC_ALL` و`LC_MESSAGES` و`LANG`. كل رسالة تعيش في فهرس، ويقع اختبار لو تركت لغة واحدة منه. الهدف كل لغة في أوراسيا، ثم أكثر.

## خارطة الطريق

1. **التكلم.** مهايئات متوافقة مع OpenAI وAnthropic، بتدفق SSE وبقراءة الأجوبة المكتملة. أدوار الوكيل بالأدوات تستخدم حالياً أجوبة مكتملة. *منجز.*
2. **الإعدادات والهيئة.** ملفات إعدادات TOML، عالمية ولكل مشروع، بـ`config set` و`get` و`unset` ووصف لكل مفتاح. واجهة بثيم في روح [Hermes Agent](https://github.com/NousResearch/hermes-agent): لافتة، ولوحة معلومات، وأجوبة مؤطرة، ودوّار، افتراضياً بلون الكِمّاث البنفسجي، وأي لون قابل للضبط بصيغة HEX. *منجز.*
3. **نماذج كثيرة.** أسماء بديلة للنماذج، وسؤال عدة نماذج في آن واحد ومقارنة الأجوبة، وسلاسل احتياط، ونموذج حكم يصهرها معاً. *منجز.*
4. **العمل.** نداءات منظمة للأدوات لمزوّدين متوافقة مع OpenAI وAnthropic، وأدوات محلية للملفات والبحث والصَدَفة، وجلب الروابط، ومهارات محلية وأذونات allow/ask/deny؛ يعرض السجل كل نداء أداة ونتيجته. *منجز.*
5. **التذكر.** المحادثات المسماة تدوم محلياً بين مرات التشغيل. الذاكرة الدلالية طويلة الأمد ما تزال أمامنا.
6. **التوسيع.** خوادم MCP، وملحقات خارجية، ووكلاء فرعيون، ودعم أغنى للمهارات.
7. **الوصول.** آلية المتصفح، وبوابات المراسلة، ومهام مجدولة.

التفكير وراء التصميم في [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## الحدود المعروفة

- منتقيات المزوّدين والنماذج تسرد المزوّدين المضبوطين والأسماء البديلة للنماذج. لا يكتشف Marelle بعد فهرسات النماذج البعيدة.
- المنتقيات تدعم نقر الفأرة للتركيز والتدحرج بالعجلة؛ السجل ومحرر الإدخال يقودان بلوحة المفاتيح.
- Ctrl-C يخرج من الجلسة التفاعلية. لا يلغي بعد فقط الجواب الجاري.
- حلقة الوكيل، وأدوات محلية واعية بالأذونات، وجلسات محلية باثقة منفذة، لكن Marelle يبقى نموذجاً أولياً قبل ألفا؛ للتصميم الحالي وحدوده اقرأ [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## التطوير

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

الاختبارات لا تلمس الشبكة أبداً: فهي تتكلم مع خادم زائف في `crates/marelle-testkit`. يبني GitHub Actions أيضاً التنفيذيات لأندرويد ويفحص أدنى إصدار Rust معلناً. لمرآة GitLab يوجد خط GitLab CI؛ انظر [docs/GITLAB.md](docs/GITLAB.md).

## الرخصة

[AGPL-3.0-only](LICENSE). إشعارات الأطراف الثالثة في [NOTICE](NOTICE). أرشيفات الإصدار تشمل `THIRD-PARTY-LICENSES.txt` لصناديق Rust داخل التنفيذي.
