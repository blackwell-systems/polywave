[English](../../README.md) · [简体中文](README.zh-CN.md) · [Русский](README.ru.md) · [हिन्दी](README.hi.md) · **العربية**

# Polywave

<p align="center">
  <img src="assets/logo.png" alt="Polywave" width="600" />
</p>

<p align="center">
  <a href="https://github.com/blackwell-systems"><img src="https://raw.githubusercontent.com/blackwell-systems/blackwell-docs-theme/main/badge-trademark.svg" alt="Blackwell Systems" /></a>
  <img src="https://img.shields.io/badge/version-0.11.0-blue" alt="Version" />
  <a href="https://agentskills.io"><img src="assets/badge-agentskills.svg" alt="Agent Skills" /></a>
  <a href="https://buymeacoffee.com/blackwellsystems"><img src="https://img.shields.io/badge/buy%20me%20a%20coffee-donate-yellow.svg" alt="Buy Me A Coffee" /></a>
</p>

**وكلاء ذكاء اصطناعي متوازون لا يُفسد أحدهم كود الآخر، داخل واجهة CLI التي تستخدمها بالفعل.**

Polywave طبقة خفيفة فوق ما لديك، وليست منصة وكلاء. تواصل عملك في Claude Code (أو Codex). ثبّته مرة واحدة، وبعدها تصبح واجهتك بالكامل هي `/polywave` داخل الأداة التي تستخدمها بالفعل: كل وكيل يحصل على worktree خاص به، وكل ملف يُسنَد إلى وكيل واحد بالضبط، وترى الخطة الكاملة قبل أن يلمس أي وكيل كودك. تُحَل التعارضات وقت التخطيط، لا وقت الدمج.

لا تتبنّى بيئة تشغيل، ولا تنتقل إلى أداة جديدة، ولا تشغّل محرك مراسلة/ذاكرة/تنسيق. يضيف التثبيت مهارة skill ومجموعة من الخطافات hooks وملف `polywave-tools` الثنائي. تقود المهارة والخطافات سير العمل: فهي تستدعي الملف الثنائي خلف الكواليس، ولذا في الاستخدام العادي تكتفي بكتابة `/polywave scout "feature"` و `/polywave wave`. تبقى واجهة CLI متاحة حين تريدها (للاسترداد، والبرمجة النصية، وCI، والاستخدام المتقدم)، لكن معظم الجلسات لا تلمسها مباشرةً. تطلب منك أطر الوكلاء الثقيلة أن تنتقل إلى عالمها لتحصل على التوازي؛ أما Polywave فيلتقي بك في عالمك أنت ويجعل الدمج آمناً.

> منشور بصيغة [Agent Skill](https://agentskills.io) (معيار مفتوح). متوافق مع Claude Code و Cursor و GitHub Copilot وأدوات أخرى متوافقة مع Agent Skills.

> **جديد على Polywave؟**
> 1. اقرأ هذا الملف README (15 دقيقة)
> 2. اقرأ [QUICKSTART.md](implementations/claude-code/QUICKSTART.md) (20 دقيقة) للاطلاع على مثال محلول
> 3. جرّبه: `/polywave scout "feature"` على مشروع تجريبي
> 4. تعمّق: [polywave-protocol](https://github.com/blackwell-systems/polywave-protocol) للمواصفة الكاملة

## لماذا

لقد شغّلت وكلاء متوازين من قبل. تعرف ما يحدث: يحرّر وكيلان الملف نفسه، فيُنتج الدمج فوضى، وتقضي في إصلاحها وقتاً أطول مما لو أنجزت العمل بالتتابع. أو أسوأ من ذلك، ينجح الدمج بصمت لأن الوكيلين عدّلا دالتين مختلفتين في الملف نفسه، لكنهما بَنَيا افتراضات متناقضة حول الحالة المشتركة. تكتشف ذلك وقت التشغيل.

تحاول معظم الأطر حلّ هذا بتحسين المطالبات (prompts). أما Polywave فيحلّه بالبنية:

- **ملكية ملفات غير متقاطعة (disjoint).** يُسنِد Scout كل ملف إلى وكيل واحد بالضبط قبل كتابة أي كود. لا يمكن لوكيلين في الموجة نفسها أن يُنتجا تعديلات على الملف نفسه. يُفرَض هذا الإسناد عند حدّ الأدوات، لا يُترَك لانضباط الوكيل؛ فتصبح المخالفات مستحيلة. وتصبح تعارضات الدمج مستحيلة بنيوياً.
- **عزل worktree لكل وكيل.** يعمل كل وكيل في worktree خاص به عبر git، أي دليل منفصل بشجرة ملفات مستقلة. لا تتسابق عمليات البناء والاختبار وكتابة ذاكرة التخزين المؤقت للأدوات المتزامنة على حالة مشتركة.
- **مراجعة بشرية قبل التنفيذ.** ترى الخطة الكاملة (إسنادات الملفات، عقود الواجهات، بنية الموجات) وتوافق عليها قبل إطلاق أي وكيل. هذه هي آخر نقطة يكون فيها تغيير المعمارية رخيصاً.
- **بوابة الملاءمة (suitability gate).** يقول Polywave "لا" حين لا يتفكّك العمل بنظافة. يمنع تقييمُ "غير ملائم" التفكيكاتِ الرديئة من التحوّل إلى إخفاقات باهظة الثمن.

Polywave ليس بيئة تشغيل وكلاء. فهو لا يوجّه المهام، ولا يدير المراسلة بين الوكلاء، ولا يحتفظ بذاكرة عابرة للجلسات. إنه بروتوكول تنسيق: قسّم العمل بأمان، وتحقّق من التقسيم، وأطلق الوكلاء باستقلالية، وادمج بشكل حتمي. يعمل الوكلاء بالتوازي لكنهم لا يتواصلون؛ تأتي الصحة من التقسيم، لا من التعاون.

هذه فئة وزن مختارة عن عمد. تجمع محرّكات الوكلاء الكاملة (Hermes وأطر الأسراب swarm وأمثالها) بيئةَ التشغيل والذاكرة والمراسلة والتنسيق في منصة تتبنّاها. لا يحمل Polywave شيئاً من ذلك. فهو يركب فوق بيئة تشغيل الوكلاء التي لديك بالفعل، ويضيف شيئاً واحداً بالضبط تفتقده تلك المنصات: تقسيمَ ملكية الملفات والدمجَ الحتمي اللذين يجعلان التعديلات المتوازية آمنة. إن كنت تريد محرّك وكلاء كاملاً، فاستخدم واحداً. وإن كنت تريد أن تشغّل واجهة CLI الحالية لديك عدة وكلاء برمجة في آنٍ واحد دون أن ينفجر الدمج، فهذا هو ما وُجد له Polywave.

يضم النظام سبعة [أدوار للمشاركين](https://github.com/blackwell-systems/polywave-protocol/blob/main/participants.md)، لكنك تتفاعل مع اثنين: **Orchestrator** (جلسة Claude Code خاصتك، تنسّق كل شيء) و **Scout** (يحلّل قاعدة الكود، ويسنِد الملفات، ويكتب الخطة). أما الخمسة الآخرون (Scaffold Agent و Wave Agents و Integration Agent و Critic Agent و Planner) فيعملون تلقائياً عند الحاجة.

## كيف

**ما الذي يحدث عندما تشغّل Polywave:**

1. تشغّل `/polywave scout "feature"` -> يحلّل Scout قاعدة الكود، ويسنِد الملفات إلى الوكلاء
2. يكتب Scout مستند IMPL (قطعة تنسيق بصيغة YAML تحدّد ملكية الملفات وعقود الواجهات وبنية الموجات) -> تراجع أنت بنية الموجات
3. تشغّل `/polywave wave` -> يُنشئ Scaffold Agent ملفات السقالة إن لزم الأمر
4. تنطلق Wave Agents بالتوازي -> يعمل كلٌّ منها في worktree معزول على ملفات غير متقاطعة
5. يدمج Orchestrator -> يشغّل الاختبارات -> ينظّف الـ worktrees

**الآليات الأساسية:**

- **Orchestrator:** وكيل تنسيق متزامن داخل جلستك. يطلق الوكلاء، ويفرض ملكية الملفات، وينفّذ إجراء الدمج، ويشغّل بوابات التحقق.

- **Scout:** وكيل غير متزامن. يحلّل قاعدة الكود، وينتج مستند IMPL مع رسم بياني للاعتماديات، وعقود واجهات، وجدول ملكية للملفات، وبنية موجات. كل ملف يُسنَد إلى وكيل واحد بالضبط.

- **Scaffold Agent:** يعمل مرة واحدة قبل Wave 1 إذا لزمت أنواع مشتركة. يُنشئ ملفات الأنواع المشتركة من عقود مستند IMPL، ويتحقق من التصريف (compilation)، ويعمل commit إلى HEAD.

- **Wave Agents:** وكلاء غير متزامنين يعملون بالتوازي. يملك كلٌّ منهم ملفات غير متقاطعة، وينفّذ مقابل عقود واجهات مجمّدة، ويشغّل بوابة التحقق، ويعمل commit لعمله، ويكتب تقرير إنجاز.

- **Integration Agent:** يعمل بعد دمج الموجة. يوصّل المخرجات الجديدة (exports) من وكلاء الموجة بكود الاستدعاء. غير قاتل؛ إن فشل التوصيل تُبلَّغ الثغرات إلى الإنسان.

يحوي البروتوكول **بوابة ملاءمة** مدمجة تجيب عن [خمسة أسئلة](https://github.com/blackwell-systems/polywave-protocol/blob/main/preconditions.md) قبل إنتاج أي مطالبات للوكلاء. إن لم تتحقق الشروط المسبقة، يُصدر scout الحالة NOT SUITABLE ويتوقف.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/diagrams/polywave-scout-wave-dark.svg">
  <img src="assets/diagrams/polywave-scout-wave-light.svg" alt="Polywave scout + wave execution flow">
</picture>

## بداية سريعة

**المتطلبات المسبقة:** Git 2.20+ و jq 1.6+ و Claude Code. يُنصَح بشدة بخادم لغة لمكدّسك (مثل `gopls` أو `rust-analyzer` أو `pyright` أو `typescript-language-server`): يستخدم الوكلاء LSP للتنقّل ويتراجعون إلى grep الأبطأ من دونه. لست بحاجة إلى polywave-protocol أو polywave-web.

```bash
# 1. Install skill files, hooks, and Agent permission
git clone https://github.com/blackwell-systems/polywave.git ~/code/polywave
~/code/polywave/install.sh    # configures Agent permission, symlinks skills, installs hooks

# 2. Install polywave-tools CLI (Homebrew — recommended)
brew install blackwell-systems/tap/polywave-tools

# 3. Initialize your project
cd your-project
polywave-tools init            # auto-detects language, build, and test commands

# 4. Verify
polywave-tools verify-install  # checks CLI, git, LSP, skill files, hooks, permissions
```

<details>
<summary>Alternative CLI install methods</summary>

**ملف ثنائي مُسبق البناء** (لا حاجة إلى سلسلة أدوات Go): نزّله من [أحدث إصدار](https://github.com/blackwell-systems/polywave-go/releases/latest) وانقله إلى `PATH` لديك.

**Go install:**
```bash
go install github.com/blackwell-systems/polywave-go/cmd/polywave-tools@latest
```
ملاحظة: يضع `go install` الملف الثنائي في `$(go env GOPATH)/bin` (عادةً `~/go/bin`). إذا أبلغ `polywave-tools` بعد ذلك عن "command not found"، فهذا يعني أن ذلك الدليل ليس على `PATH` لديك — أضفه:
```bash
echo 'export PATH="$(go env GOPATH)/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```
تُبلِّغ الملفات الثنائية المثبّتة بهذه الطريقة عن إصدارها بأنه `dev`؛ استخدم Homebrew أو ملفاً ثنائياً من الإصدارات للحصول على بناء يحمل رقم إصدار.
</details>

**5. أعد تشغيل Claude Code**، ثم شغّل أول scout لك:

```bash
/polywave scout "add a caching layer to the API client"
```

**الأوامر الفرعية:**

| Command | Purpose |
|---------|---------|
| `/polywave scout "<feature>"` | تحليل قاعدة الكود، وإنتاج مستند IMPL |
| `/polywave wave` | تنفيذ الموجة المعلّقة التالية |
| `/polywave wave --auto` | تنفيذ كل الموجات المتبقية دون إشراف |
| `/polywave auto "<feature>"` | Scout + التأكيد + wave في أمر واحد |
| `/polywave status` | عرض الموجة الحالية وتقدّم الوكلاء |
| `/polywave bootstrap "<project>"` | تصميم بنية مشروع جديد من الصفر |
| `/polywave interview "<description>"` | جمع متطلبات مُنظَّم |
| `/polywave program --impl <slug> ...` | تجميع مستندات IMPL المصطفّة في برنامج متوازٍ |
| `/polywave program plan/execute/status/replan` | تخطيط متعدّد الميزات وتنفيذ مُبَوَّب حسب الطبقات |
| `/polywave amend --add-wave/--redirect-agent/--extend-scope` | تعديل مستند IMPL النشِط |

**أول مرة تستخدم Polywave؟** راجع [QUICKSTART.md](implementations/claude-code/QUICKSTART.md) للحصول على إرشاد خطوة بخطوة مع مثال للمخرجات.

## المستودعات

| Repository | Purpose |
|-----------|---------|
| [polywave-protocol](https://github.com/blackwell-systems/polywave-protocol) | مواصفة البروتوكول: الثوابت، وقواعد التنفيذ، وآلة الحالات، وصيغ الرسائل |
| **polywave** (هذا المستودع) | تنفيذ Claude Code: Agent Skill، والخطافات، والمطالبات، وقوالب الوكلاء |
| [polywave-go](https://github.com/blackwell-systems/polywave-go) | محرّك Go، و Protocol SDK، وواجهة `polywave-tools` CLI |
| [polywave-web](https://github.com/blackwell-systems/polywave-web) | واجهة الويب وخادم HTTP/SSE |

## متى تستخدمه

يسدّد Polywave ثمنه حين يكون للعمل خطوط فصل واضحة بين الملفات، وتمكن معه تعريف الواجهات قبل بدء التنفيذ، ويملك كل وكيل عملاً كافياً يبرّر تشغيله بالتوازي. وكون دورة البناء/الاختبار تتجاوز 30 ثانية يضخّم الوفورات أكثر.

إن لم يتفكّك العمل بنظافة، فسيقول Scout ذلك. فهو يشغّل بوابة ملاءمة أولاً ويُصدر NOT SUITABLE بدلاً من فرض تفكيك رديء.

## كيف تعمل السلامة المتوازية

يفرض Polywave قيدين مستقلين يجعلان معاً التنفيذ المتوازي صحيحاً:

**ملكية الملفات غير المتقاطعة** تمنع تعارضات الدمج. كل ملف سيتغيّر يُسنَد إلى وكيل واحد بالضبط في مستند IMPL. لا يمكن لوكيلين في الموجة نفسها أن يُنتجا تعديلات على الملف نفسه، ولذا تكون خطوة الدمج خالية من التعارض دائماً.

**عزل worktree** يمنع التداخل وقت التنفيذ. يعمل كل وكيل في worktree خاص به عبر git، أي دليل منفصل يشارك تاريخ git نفسه لكن بشجرة ملفات مستقلة. لا تتسابق عمليات البناء والاختبار وكتابة ذاكرة التخزين المؤقت للأدوات المتزامنة على حالة مشتركة.

لا يغني أيٌّ من القيدين عن الآخر. الملكية غير المتقاطعة دون worktrees: الدمج آمن، لكن عمليات البناء المتزامنة متقلّبة. الـ worktrees دون ملكية غير متقاطعة: التنفيذ نظيف، لكن الدمج يُنتج تعارضات غير قابلة للحل. يجب أن يتحقّق كلاهما.

### دفاع عزل worktree (6 طبقات)

لا يحترم الوكلاء دائماً تعليمات العزل. يعامل Polywave عزل worktree بوصفه مشكلة بنية تحتية لا مشكلة تعاون، مع الإنفاذ القائم على الخطافات (E43) بوصفه الآلية الأساسية.

| Layer | Mechanism | Type |
|-------|-----------|------|
| **E43** | **إنفاذ قائم على الخطافات**: تقوم خطافات دورة حياة Claude Code (SubagentStart و PreToolUse:Bash و PreToolUse:Write/Edit و SubagentStop) تلقائياً بحقن متغيّرات البيئة، وإضافة أوامر cd إلى بداية استدعاءات bash، وحجب الكتابات خارج الحدود عند حدّ الأدوات. تصبح المخالفات مستحيلة لا مجرد قابلة للاكتشاف. | **الوقاية (الأساسية)** |
| 0 | **خطاف pre-commit**: يُثبَّت تلقائياً بواسطة `polywave-tools create-worktrees`. يحجب عمليات commit إلى main أثناء الموجات النشطة. يتجاوزه Orchestrator عبر `POLYWAVE_ALLOW_MAIN_COMMIT=1`. | الوقاية |
| 1 | **إنشاء الـ worktrees مسبقاً يدوياً**: يُنشئ Orchestrator كل الـ worktrees قبل إطلاق أي وكيل | حتمي |
| 2 | **المعامل `isolation: "worktree"`**: يحدّد كل إطلاق وكيل عزل worktree على مستوى الأداة | على مستوى الأداة |
| 3 | **تحقّق ذاتي في الحقل Field 0**: يؤكّد الوكلاء الفرع عبر فحص موجز (الإنفاذ الأساسي هو خطاف `validate_worktree_isolation` عند SubagentStart) | تعاوني |
| 4 | **سلك تعثّر وقت الدمج**: يَعُدّ Orchestrator عمليات commit لكل فرع worktree قبل الدمج. صفر عمليات commit = فشل عزل. يتوقّف مع خيارات الاسترداد. | حتمي |

## Polywave-Teams (تجريبي)

[`docs/proposals/polywave-teams/`](docs/proposals/polywave-teams/) طبقة تنفيذ بديلة تستخدم Claude Code Agent Teams. البروتوكول نفسه، ومستند IMPL نفسه، و Scout نفسه. لكن سباكة الموجات مختلفة: يحلّ زملاء الفريق محلّ استدعاءات أداة Agent في الخلفية، فيوفّرون المراسلة بين الوكلاء وتنبيهات الانحراف في الوقت الفعلي.

## تدوينة المدوّنة

سلسلة من أربعة أجزاء حول النمط، والدروس المستفادة من تجريبه على أنفسنا، وكيف تطوّر البروتوكول:

1. [Polywave: A Coordination Pattern for Parallel AI Agents](https://blog.blackwell-systems.com/posts/scout-and-wave/). النمط: أنماط فشل التوازي الساذج، ومُخرَج scout، وتنفيذ الموجات، ومثال محلول من brewprune.
2. [Polywave, Part 2: What Dogfooding Taught Us](https://blog.blackwell-systems.com/posts/scout-and-wave-part2/). حلقة التدقيق-الإصلاح-التدقيق، وقياس النفقات الإضافية (أبطأ بنسبة 88% عند تجاهله)، ووضع Quick، ومشكلة الإقلاع (bootstrap) للمشاريع الجديدة.
3. [Polywave, Part 3: Five Failures, Five Fixes](https://blog.blackwell-systems.com/posts/scout-and-wave-part3/). كيف تفكّك ملف المهارة من كتلة أحادية من 400 سطر، ولماذا تهمّ ترويسات الإصدارات، وخمسة إصلاحات لمطالبة scout مدفوعة بإخفاقات حقيقية.
4. [Polywave, Part 4: Trust Is Structural](https://blog.blackwell-systems.com/posts/scout-and-wave-part4/). Scaffold Agent، ودفاع عزل worktree ذو الطبقات الخمس، ولماذا تنتمي الصحة إلى البنية التحتية لا إلى التعاون.

## الترخيص

[MIT OR Apache-2.0](LICENSE)
