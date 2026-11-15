```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**סוכן בינה מלאכותית ניתן להגדרה עבור המסוף שלך.**
הבא מפתחות API משלך: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, Ollama מקומי, או כל שרת תואם OpenAI. קודם כל לינוקס ו-Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · עברית · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **טרום-אלפא, גרסה 0.1.0-alpha.1.** ממומש: ממשק TUI ילידי, צ׳אט עם כמה ספקים, לולאת סוכן מודעת-הרשאות, כלים מקומיים והפעלות מקומיות שניתן לחזור אליהן. ממשקי ה-API וההגדרות עשויים עדיין להשתנות. זהו אב-טיפוס, ולא שוויון יכולות עם OpenCode או Hermes.

*Marelle* הוא השם הצרפתי של משחק הקפיצות בין המשבצות. כל משבצת היא צעד.

## התקנה

**Termux באנדרואיד.** הורד בינארי מוכן מדף [המהדורות](https://github.com/S-I-L-S/marelle/releases), או בנה אותו בטלפון. מדריך צעד אחר צעד: [docs/TERMUX.md](docs/TERMUX.md).

**לינוקס וכל מקום שבו Rust רץ.** נדרש Rust 1.85 או חדש יותר:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

או שכפל את המאגר והרץ `cargo build --release`. הבינארי הוא `target/release/marelle`, בסביבות 4 MiB.

## הצ׳אט הראשון

שמור את המפתח בקובץ שרק אתה קורא, והודע ל-Marelle היכן הוא. Marelle לעולם לא שומר מפתח בהגדרות שלו.

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

החלף את `MODEL` בשם מודל מהרשימה של הספק שלך. דרכי שימוש אחרות:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

התשובה הולכת ל-stdout. הרהורים, ספירת אסימונים, הערות ושגיאות הולכים ל-stderr, כך ש-`marelle chat ... > answer.txt` שומר רק את התשובה.

## כלי הסוכן

Marelle שולח הגדרות מובנות של כלים דרך ממשקי API תואמי OpenAI ו-Anthropic. ערכת הכלים המובנית:

| כלי | מטרה |
| --- | --- |
| `terminal` | הרצת פקודת מעטפת מתיקיית ההפעלה |
| `read_file`, `write_file`, `edit_file` | קריאה ועריכה של קובצי טקסט UTF-8 |
| `list_directory`, `search_text` | עיון וחיפוש בקובצי הפרויקט |
| `fetch_url` | אחזור עמוד HTTP(S) ציבורי כטקסט (דורש `curl`) |
| `list_skills`, `load_skill` | גילוי וטעינה של הנחיות `SKILL.md` מקומיות |

הפקודה `/tools` מציגה את רשימת הכלים ו-`/skills` מפרטת את הכישורים הזמינים. הכוונה של הפרויקט נטענת מ-`AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` וקובצי `.marelle/` נבחרים כשהם בנתיבים מותרים.

כלי הקבצים מוגבלים ל-`permissions.allowed_paths` (ברירת מחדל: תיקיית הפרויקט הנוכחית) ומכבדים את `permissions.denied_paths`. קריאה מותרת כברירת מחדל; שינוי קבצים, פקודות מעטפת ואחזורי רשת שואלים קודם. הרשאת המעטפת חלה על פקודות במחשב המארח ואינה ארגז חול של מערכת הקבצים: השאר אותה על `ask`, אלא אם כן אתה מאמין במודל להרצת פקודות במודע.

הסוכן עוצר לאחר `agent.max_turns` סבבי כלים. תשובות כלים נחשבות לתוכן לא אמין. הבקשות משתמשות בתשובות מלאות של המודל, כך שארגומנטים של כלים לא מורכבים משברי זרם חסרים.

## סשנים

שיחות נשמרות מקומית כברירת מחדל, לרבות היסטוריית קריאות כלים והגדרות מודל וסשן. ביוניקס לספריית הסשנים מצב `0700` ולקובץ ה-JSON מצב `0600`. הקובץ נשמר ב-`$XDG_STATE_HOME/marelle/sessions.json`, או ב-`~/.local/state/marelle/sessions.json` כאשר `XDG_STATE_HOME` אינו מוגדר. מפתחות API לעולם לא נכתבים בקובץ זה. כבה את השמירה ב:

```sh
marelle config set agent.persist_sessions false
```

קודי יציאה: `0` הצלחה, `1` שגיאת ספק או רשת, `2` שגיאת שימוש או הגדרה.

## ספקים

| id | פרוטוקול | משתנה המפתח | |
| --- | --- | --- | --- |
| `zai` | תואם OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), בינלאומי |
| `zai_coding` | תואם OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, נקודת קצה תואמת Anthropic |
| `zhipu` | תואם OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, סין |
| `zhipu_coding` | תואם OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, סין |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | תואם OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | תואם OpenAI | `OPENROUTER_API_KEY` | מודלים רבים מאחורי מפתח אחד |
| `ollama` | תואם OpenAI | אין | Ollama במכונה הזאת |

נקודות הקצה של Z.ai ו-Zhipu מגיעות מתיעוד של צדדים שלישיים, לא מהדפים של הספק עצמו. בדוק אותן מול התוכנית שלך וגבר על כל שדה אם הן שונות.

כל שדה של ספק ניתן לשינוי, וספק חדש הוא פשוט שם עם כתובת:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

שדות ספק אחרים: `kind` (`openai-compatible` או `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` ו-`extra_body`. אם המשתנה המכונה ב-`api_key_env` מוגדר, הוא גובר; אחרת הקובץ נקרא.

## הגדרות

הגדרות באות בשכבות: ערכי ברירת מחדל מובנים, אחר כך משתני סביבה, אחר כך דגלים. קובצי הגדרות נדחקים בין ברירות המחדל לסביבה.

- `MARELLE_CHAT__MODEL=zai/MODEL` מגדיר את `chat.model`. קו תחתון כפול עומד במקום נקודה.
- `marelle config show [prefix]` מפרט הגדרות ואת השכבה ממנה כל אחת באה.
- `marelle config explain <key>` מציג כל שכבה שהגדירה מפתח, מהעדיפות הנמוכה לגבוהה.

| מפתח | ברירת מחדל | מובן |
| --- | --- | --- |
| `chat.provider`, `chat.model` | אין | עם מי לדבר. `chat.model` יכול להיות `provider/model`. |
| `chat.system` | אין | בקשת מערכת |
| `chat.max_tokens`, `chat.temperature` | אין | לא מוגדר — מושאר לספק |
| `chat.stream` | `true` | זרימת תשובת השופט האופציונלי; סבבי סוכן עם כלים כרגע משתמשים בתשובות מלאות, כדי שקריאות כלים יישארו שלמות |
| `chat.show_reasoning`, `chat.show_usage` | `false` | הרהורים וספירת אסימונים ב-stderr |
| `agent.max_turns` | `20` | מקסימום סבבי כלים לכל בקשת משתמש |
| `agent.persist_sessions` | `true` | שמירת שיחות מקומיות בין הפעלות |
| `permissions.shell` | `ask` | הרצת פקודות מעטפת: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | קריאת קבצים: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | שינוי קבצים: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | אחזור URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | שורשים מותרים לכלי הקבצים |
| `permissions.denied_paths` | אין | נתיבי קבצים חסומים במפורש |
| `network.connect_timeout_secs` | `15` | הזמן המותר להתחברות |
| `network.idle_timeout_secs` | `300` | השתיקה המותרת בהמתנה לנתונים |
| `network.proxy` | `system` | `system`, `none`, או כתובת מתווך |
| `network.ca_bundle` | אין | קובץ נוסף של רשויות תעוד מהימנות |
| `network.allow_insecure_http` | `false` | לאפשר מפתח על גבי `http` חשוף אל מארח זר |
| `update.check` | `true` | לשאול את GitHub ו-GitLab על גרסה חדשה יותר בפתיחת סשן אינטראקטיבי |
| `update.github_repo` | `S-I-L-S/marelle` | `owner/repo` ב-GitHub שממנו הבדיקה שואלת את המהדורה החדשה ביותר |
| `update.gitlab_repo` | אין | `owner/repo` ב-GitLab לאותה בדיקה. ריק מכבה את GitLab |
| `update.timeout_secs` | `3` | כמה שניות הבדיקה רשאית להקצות למקור אחד |

Marelle נושא עמו רשימה משלו של רשויות תעוד מהימנות, ולכן עובד גם במקום שבו למערכת אין, כמו ב-Termux. `SSL_CERT_FILE` מוסיף עוד.

כשנמצאת גרסה חדשה יותר, הסשן מזכיר אותה פעם אחת — את הגרסה, את מה שיש לך, ומאין לקחת. מהדורה נשאלת ראשונה; מאגר שרק חותם את ה-commit שלו בתגים נותן את התג החדש ביותר. כל דבר שנכשל נשאר שקט.

## המפתחות שלך נשארים שלך

- ההגדרות מציינות משתנה או קובץ שבו המפתח שוכן, לעולם לא את המפתח עצמו. הגדרה כמו `providers.x.api_key` נדחית.
- את המפתח לא מדפיסים `config show`, `config explain` או `providers`, והוא נגזר מכל טקסט שגיאה שחוזר משרת.
- הוא נשלח רק לכתובת הספק, בכותרת אחת, ולעולם לא על גבי `http` חשוף אל מכונה אחרת, אלא אם נתת לכך היתר.
- מעקפים אינם נעקבים, כך שמפתח לא יכול להישלח הלאה למקום אחר.

## שפות

הממשק מדבר אנגלית, רוסית, צרפתית ואוקראינית. בחר ב-`--lang <code>` או ב-`MARELLE_LANG`; אחרת מכריעים `LC_ALL`, `LC_MESSAGES` ו-`LANG`. כל הודעה חיה בקטלוג, ובדיקה נופלת אם שפה כלשהי משמיטה אחת. המטרה: כל שפה של אירואסיה, ואז עוד.

## מפת הדרכים

1. **לדבר.** מתאמים תואמי OpenAI ו-Anthropic, הן עם זרימת SSE והן עם פענוח תשובות שלמות. סבבי סוכן עם כלים כרגע משתמשים בתשובות מלאות. *הושלם.*
2. **הגדרות ומראה.** קובצי הגדרות TOML, גלובליים ולפי פרויקט, עם `config set`, `get` ו-`unset` ותיאור לכל מפתח. ממשק בעל תמה ברוח [Hermes Agent](https://github.com/NousResearch/hermes-agent): באנר, פאנל מידע, תשובות במסגרת, מחוין, כברירת מחדל בסגול הוויסטריה, כל צבע ניתן להגדרה כ-HEX. *הושלם.*
3. **מודלים רבים.** כינויי מודלים, שאילת כמה מודלים בו-זמנית והשוואת התשובות, שרשראות גיבוי, ומודל שופט המותך אותן יחד. *הושלם.*
4. **לפעול.** קריאות כלים מובנות לספקים תואמי OpenAI ו-Anthropic, כלים מקומיים לקבצים/חיפוש/טרמינל, אחזור URL, כישורים מקומיים והרשאות allow/ask/deny; התמלול מציג כל קריאת כלי ואת תוצאתה. *הושלם.*
5. **לזכור.** שיחות מכוננות שורדות מקומית בין הפעלות. זיכרון סמנטי לטווח ארוך עוד לפנינו.
6. **להרחיב.** שרתי MCP, תוספים חיצוניים, תת-סוכנים ותמיכה עשירה יותר בכישורים.
7. **להגיע.** אוטומציה של דפדפן, שערי שליחות ומשימות מתוכננות.

ההיגיון מאחורי העיצוב נמצא ב-[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## מגבלות ידועות

- בוררי הספקים והמודלים מפרטים ספקים שהוגדרו וכינויי מודלים. Marelle עדיין אינו מגלה קטלוגי מודלים מרוחקים.
- הבוררים תומכים בלחיצת עכבר למיקוד ובגלילה בגלגלת; התמלול ועורך הקלט נותרים מונהגים במקלדת.
- Ctrl-C יוצא מהסשן האינטראקטיבי. הוא עדיין לא מבטל רק את התשובה הנוכחית.
- לולאת הסוכן, כלים מקומיים מודעי-הרשאות וסשנים מקומיים עמידים ממומשים, אך Marelle נשאר אב-טיפוס טרום-אלפא; לעיצוב הנוכחי ולגבולותיו קרא [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## פיתוח

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

הבדיקות לעולם אינן נוגעות לרשת: הן מדברות עם שרת מדומה ב-`crates/marelle-testkit`. GitHub Actions בונה גם את הבינאריים לאנדרואיד ומאמת את גרסת ה-Rust המינימלית המוצהרת. למראה ב-GitLab יש צינור GitLab CI; ראה [docs/GITLAB.md](docs/GITLAB.md).

## רישיון

[AGPL-3.0-only](LICENSE). הודעות צד שלישי ב-[NOTICE](NOTICE). ארכיוני המהדורות כוללים `THIRD-PARTY-LICENSES.txt` עבור תיבות ה-Rust בתוך הבינארי.
