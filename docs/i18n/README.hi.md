[English](../../README.md) · [简体中文](README.zh-CN.md) · [Русский](README.ru.md) · **हिन्दी** · [العربية](README.ar.md)

# Polywave

<p align="center">
  <img src="../../assets/logo.png" alt="Polywave" width="600" />
</p>

<p align="center">
  <a href="https://github.com/blackwell-systems"><img src="https://raw.githubusercontent.com/blackwell-systems/blackwell-docs-theme/main/badge-trademark.svg" alt="Blackwell Systems" /></a>
  <img src="https://img.shields.io/badge/version-0.11.0-blue" alt="Version" />
  <a href="https://agentskills.io"><img src="../../assets/badge-agentskills.svg" alt="Agent Skills" /></a>
  <a href="https://buymeacoffee.com/blackwellsystems"><img src="https://img.shields.io/badge/buy%20me%20a%20coffee-donate-yellow.svg" alt="Buy Me A Coffee" /></a>
</p>

**समानांतर AI एजेंट जो एक-दूसरे का कोड नहीं तोड़ते, उसी CLI में जिसे आप पहले से इस्तेमाल करते हैं।**

Polywave एक हल्की ओवरले परत है, कोई एजेंट प्लेटफ़ॉर्म नहीं। आप Claude Code (या Codex) में काम करते रहते हैं। एक बार इंस्टॉल करें, फिर आपका पूरा इंटरफ़ेस उसी टूल में `/polywave` है जिसे आप पहले से इस्तेमाल करते हैं: हर एजेंट को अपना खुद का worktree मिलता है, हर फ़ाइल ठीक एक ही एजेंट को सौंपी जाती है, और किसी भी एजेंट के आपके कोड को छूने से पहले आप पूरी योजना देख लेते हैं। टकराव विलय (merge) के समय नहीं, बल्कि योजना बनाने के समय हल हो जाते हैं।

आपको न किसी रनटाइम को अपनाना है, न किसी नए टूल पर माइग्रेट करना है, न ही कोई मैसेजिंग/मेमोरी/ऑर्केस्ट्रेशन इंजन चलाना है। इंस्टॉल एक skill, hook का एक सेट, और `polywave-tools` बाइनरी जोड़ता है। skill और hook वर्कफ़्लो को चलाते हैं: वे परदे के पीछे बाइनरी को कॉल करते हैं, इसलिए सामान्य उपयोग में आप बस `/polywave scout "feature"` और `/polywave wave` टाइप करते हैं। जब आप चाहें तब CLI मौजूद है (रिकवरी, स्क्रिप्टिंग, CI, पावर उपयोग के लिए), लेकिन अधिकांश सत्र इसे सीधे कभी नहीं छूते। भारी-भरकम एजेंट फ़्रेमवर्क आपसे समानांतरता पाने के लिए उनकी दुनिया में शिफ़्ट होने को कहते हैं; Polywave आपसे आपकी अपनी दुनिया में मिलता है और विलय को सुरक्षित बनाता है।

> एक [Agent Skill](https://agentskills.io) (खुला मानक) के रूप में प्रकाशित। Claude Code, Cursor, GitHub Copilot और अन्य Agent Skills-संगत टूल्स के साथ संगत।

> **Polywave में नए हैं?**
> 1. यह README पढ़ें (15 मिनट)
> 2. एक हल किए गए उदाहरण के लिए [QUICKSTART.md](../../implementations/claude-code/QUICKSTART.md) पढ़ें (20 मिनट)
> 3. आज़माएँ: किसी टेस्ट प्रोजेक्ट पर `/polywave scout "feature"`
> 4. गहराई में जाएँ: पूरी विशिष्टि के लिए [polywave-protocol](https://github.com/blackwell-systems/polywave-protocol)

## क्यों

आपने पहले भी समानांतर एजेंट चलाए हैं। आप जानते हैं क्या होता है: दो एजेंट एक ही फ़ाइल को संपादित करते हैं, विलय कचरा पैदा करता है, और आप इसे ठीक करने में उससे ज़्यादा समय लगा देते हैं जितना क्रमिक रूप से काम करने में लगता। या इससे भी बुरा, विलय चुपचाप सफल हो जाता है क्योंकि दोनों एजेंटों ने एक ही फ़ाइल में अलग-अलग फ़ंक्शन छुए, पर उन्होंने साझा स्थिति (shared state) के बारे में परस्पर विरोधी धारणाएँ बना लीं। आपको यह रनटाइम पर पता चलता है।

अधिकांश फ़्रेमवर्क इसे बेहतर प्रॉम्प्ट से हल करने की कोशिश करते हैं। Polywave इसे संरचना से हल करता है:

- **असंयुक्त (disjoint) फ़ाइल स्वामित्व।** कोई भी कोड लिखे जाने से पहले Scout हर फ़ाइल को ठीक एक एजेंट को सौंप देता है। एक ही wave में दो एजेंट एक ही फ़ाइल में संपादन नहीं कर सकते। यह असाइनमेंट एजेंट के अनुशासन पर नहीं छोड़ा जाता, बल्कि टूल की सीमा पर लागू किया जाता है; उल्लंघन असंभव हो जाते हैं। विलय टकराव संरचनात्मक रूप से असंभव हो जाते हैं।
- **प्रति-एजेंट worktree पृथक्करण।** हर एजेंट अपने खुद के git worktree में काम करता है, एक अलग निर्देशिका जिसका स्वतंत्र फ़ाइल-वृक्ष होता है। समवर्ती बिल्ड, टेस्ट और टूल-कैश राइट साझा स्थिति पर टकराते नहीं।
- **निष्पादन से पहले मानवीय समीक्षा।** आप पूरी योजना देखते हैं (फ़ाइल असाइनमेंट, इंटरफ़ेस अनुबंध, wave संरचना) और किसी एजेंट के शुरू होने से पहले उसे मंज़ूरी देते हैं। यह वह आख़िरी बिंदु है जहाँ आर्किटेक्चर बदलना सस्ता है।
- **उपयुक्तता द्वार (suitability gate)।** जब काम साफ़-साफ़ विघटित नहीं होता, Polywave "नहीं" कह देता है। एक ख़राब-मेल वाला आकलन ख़राब विघटन को महँगी विफलताओं में बदलने से रोकता है।

Polywave कोई एजेंट रनटाइम नहीं है। यह न कार्यों को रूट करता है, न एजेंटों के बीच मैसेजिंग संभालता है, न सत्रों के आर-पार मेमोरी रखता है। यह एक समन्वय प्रोटोकॉल है: काम को सुरक्षित रूप से बाँटें, बँटवारे की पुष्टि करें, एजेंटों को स्वतंत्र रूप से शुरू करें, नियतात्मक रूप से (deterministically) विलय करें। एजेंट समानांतर चलते हैं पर संवाद नहीं करते; शुद्धता बँटवारे से आती है, सहयोग से नहीं।

यह एक जानबूझकर चुना गया भार-वर्ग है। पूर्ण एजेंट इंजन (Hermes, swarm फ़्रेमवर्क, और इनके जैसे) निष्पादन रनटाइम, मेमोरी, मैसेजिंग और ऑर्केस्ट्रेशन को एक प्लेटफ़ॉर्म में बाँध देते हैं जिसे आप अपनाते हैं। Polywave इनमें से कुछ भी नहीं ढोता। यह उस एजेंट रनटाइम पर सवारी करता है जो आपके पास पहले से है और ठीक एक चीज़ जोड़ता है जो उन प्लेटफ़ॉर्मों में नहीं है: फ़ाइल-स्वामित्व का बँटवारा और नियतात्मक विलय जो समानांतर संपादनों को सुरक्षित बनाते हैं। यदि आपको पूरा एजेंट इंजन चाहिए, तो एक इस्तेमाल कीजिए। यदि आप चाहते हैं कि आपका मौजूदा CLI एक साथ कई कोडिंग एजेंट चलाए बिना विलय के बिखरे, तो Polywave इसी के लिए है।

तंत्र में सात [प्रतिभागी भूमिकाएँ](https://github.com/blackwell-systems/polywave-protocol/blob/main/participants.md) हैं, पर आप दो के साथ अंतःक्रिया करते हैं: **Orchestrator** (आपका Claude Code सत्र, सब कुछ समन्वित करता है) और **Scout** (कोडबेस का विश्लेषण करता है, फ़ाइलें सौंपता है, योजना लिखता है)। बाकी पाँच (Scaffold Agent, Wave Agents, Integration Agent, Critic Agent, Planner) ज़रूरत पड़ने पर अपने आप चलते हैं।

## कैसे

**जब आप Polywave चलाते हैं तब क्या होता है:**

1. आप `/polywave scout "feature"` चलाते हैं -> Scout कोडबेस का विश्लेषण करता है, एजेंटों को फ़ाइलें सौंपता है
2. Scout एक IMPL दस्तावेज़ लिखता है (एक YAML समन्वय आर्टिफ़ैक्ट जो फ़ाइल स्वामित्व, इंटरफ़ेस अनुबंध और wave संरचना परिभाषित करता है) -> आप wave संरचना की समीक्षा करते हैं
3. आप `/polywave wave` चलाते हैं -> ज़रूरत होने पर Scaffold Agent स्कैफ़ोल्ड फ़ाइलें बनाता है
4. Wave Agents समानांतर में शुरू होते हैं -> हर एक असंयुक्त फ़ाइलों पर एक पृथक worktree में काम करता है
5. Orchestrator विलय करता है -> टेस्ट चलाता है -> worktree साफ़ करता है

**मुख्य तंत्र:**

- **Orchestrator:** आपके सत्र में एक तुल्यकालिक (synchronous) समन्वय एजेंट। एजेंट शुरू करता है, फ़ाइल स्वामित्व लागू करता है, विलय प्रक्रिया निष्पादित करता है, सत्यापन द्वार चलाता है।

- **Scout:** एक अतुल्यकालिक (asynchronous) एजेंट। कोडबेस का विश्लेषण करता है, निर्भरता ग्राफ़, इंटरफ़ेस अनुबंध, फ़ाइल स्वामित्व तालिका और wave संरचना के साथ IMPL दस्तावेज़ बनाता है। हर फ़ाइल ठीक एक एजेंट को सौंपी जाती है।

- **Scaffold Agent:** यदि साझा प्रकारों की ज़रूरत हो तो Wave 1 से पहले एक बार चलता है। IMPL दस्तावेज़ के अनुबंधों से साझा प्रकार फ़ाइलें बनाता है, संकलन की पुष्टि करता है, HEAD पर कमिट करता है।

- **Wave Agents:** समानांतर में चलने वाले अतुल्यकालिक एजेंट। हर एक असंयुक्त फ़ाइलों का स्वामी होता है, जमे हुए (frozen) इंटरफ़ेस अनुबंधों के विरुद्ध कार्यान्वयन करता है, सत्यापन द्वार चलाता है, काम कमिट करता है, समाप्ति रिपोर्ट लिखता है।

- **Integration Agent:** wave विलय के बाद चलता है। wave एजेंटों से नए एक्सपोर्ट को कॉलर कोड में जोड़ता है। घातक नहीं; यदि जोड़ना विफल होता है, तो अंतराल मनुष्य को रिपोर्ट किए जाते हैं।

प्रोटोकॉल में एक अंतर्निहित **उपयुक्तता द्वार** है जो किसी भी एजेंट प्रॉम्प्ट को बनाने से पहले [पाँच प्रश्नों](https://github.com/blackwell-systems/polywave-protocol/blob/main/preconditions.md) का उत्तर देता है। यदि पूर्वशर्तें नहीं टिकतीं, तो scout NOT SUITABLE जारी करके रुक जाता है।

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/diagrams/polywave-scout-wave-dark.svg">
  <img src="../../assets/diagrams/polywave-scout-wave-light.svg" alt="Polywave scout + wave execution flow">
</picture>

## त्वरित शुरुआत

**पूर्वापेक्षाएँ:** Git 2.20+, jq 1.6+, Claude Code। आपके स्टैक के लिए एक भाषा सर्वर (जैसे `gopls`, `rust-analyzer`, `pyright`, `typescript-language-server`) की पुरज़ोर सिफ़ारिश की जाती है: एजेंट नेविगेशन के लिए LSP का उपयोग करते हैं और इसके बिना धीमे grep पर लौट जाते हैं। आपको polywave-protocol या polywave-web की ज़रूरत नहीं है।

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

**पूर्व-निर्मित बाइनरी** (Go टूलचेन की ज़रूरत नहीं): [नवीनतम रिलीज़](https://github.com/blackwell-systems/polywave-go/releases/latest) से डाउनलोड करें और इसे अपने `PATH` पर ले जाएँ।

**Go install:**
```bash
go install github.com/blackwell-systems/polywave-go/cmd/polywave-tools@latest
```
ध्यान दें: `go install` बाइनरी को `$(go env GOPATH)/bin` (आमतौर पर `~/go/bin`) में रखता है। यदि बाद में `polywave-tools` "command not found" बताता है, तो वह निर्देशिका आपके `PATH` पर नहीं है — इसे जोड़ें:
```bash
echo 'export PATH="$(go env GOPATH)/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```
इस तरह इंस्टॉल की गई बाइनरी अपना संस्करण `dev` के रूप में बताती हैं; संस्करण-सहित बिल्ड के लिए Homebrew या रिलीज़ बाइनरी का उपयोग करें।
</details>

**5. Claude Code को पुनः आरंभ करें**, फिर अपना पहला scout चलाएँ:

```bash
/polywave scout "add a caching layer to the API client"
```

**उपकमांड:**

| Command | Purpose |
|---------|---------|
| `/polywave scout "<feature>"` | कोडबेस का विश्लेषण करें, IMPL दस्तावेज़ बनाएँ |
| `/polywave wave` | अगला लंबित wave निष्पादित करें |
| `/polywave wave --auto` | शेष सभी wave बिना निगरानी के निष्पादित करें |
| `/polywave auto "<feature>"` | एक ही कमांड में Scout + पुष्टि + wave |
| `/polywave status` | वर्तमान wave और एजेंट प्रगति दिखाएँ |
| `/polywave bootstrap "<project>"` | शून्य से नई प्रोजेक्ट संरचना डिज़ाइन करें |
| `/polywave interview "<description>"` | संरचित आवश्यकता संग्रहण |
| `/polywave program --impl <slug> ...` | कतारबद्ध IMPL को एक समानांतर प्रोग्राम में बंडल करें |
| `/polywave program plan/execute/status/replan` | बहु-फ़ीचर योजना और स्तर-गेटेड निष्पादन |
| `/polywave amend --add-wave/--redirect-agent/--extend-scope` | सक्रिय IMPL को संशोधित करें |

**पहली बार Polywave का उपयोग कर रहे हैं?** उदाहरण आउटपुट के साथ चरण-दर-चरण मार्गदर्शन के लिए [QUICKSTART.md](../../implementations/claude-code/QUICKSTART.md) देखें।

## रिपॉज़िटरीज़

| Repository | Purpose |
|-----------|---------|
| [polywave-protocol](https://github.com/blackwell-systems/polywave-protocol) | प्रोटोकॉल विशिष्टि: अपरिवर्तनीय (invariants), निष्पादन नियम, स्टेट मशीन, संदेश प्रारूप |
| **polywave** (यह रिपॉज़िटरी) | Claude Code कार्यान्वयन: Agent Skill, hook, प्रॉम्प्ट, एजेंट टेम्पलेट |
| [polywave-go](https://github.com/blackwell-systems/polywave-go) | Go इंजन, Protocol SDK, और `polywave-tools` CLI |
| [polywave-web](https://github.com/blackwell-systems/polywave-web) | वेब UI और HTTP/SSE सर्वर |

## इसे कब इस्तेमाल करें

Polywave तब अपनी कीमत वसूल करता है जब काम में स्पष्ट फ़ाइल सीवन (seam) हों, कार्यान्वयन शुरू होने से पहले इंटरफ़ेस परिभाषित किए जा सकें, और हर एजेंट के पास समानांतर चलने को उचित ठहराने लायक पर्याप्त काम हो। बिल्ड/टेस्ट चक्र का 30 सेकंड से ज़्यादा होना बचत को और बढ़ा देता है।

यदि काम साफ़-साफ़ विघटित नहीं होता, तो Scout ऐसा कह देता है। यह पहले एक उपयुक्तता द्वार चलाता है और किसी ख़राब विघटन को थोपने के बजाय NOT SUITABLE जारी करता है।

## समानांतर सुरक्षा कैसे काम करती है

Polywave दो स्वतंत्र बाध्यताओं को लागू करता है जो मिलकर समानांतर निष्पादन को सही बनाती हैं:

**असंयुक्त फ़ाइल स्वामित्व** विलय टकराव रोकता है। जो भी फ़ाइल बदलेगी वह IMPL दस्तावेज़ में ठीक एक एजेंट को सौंपी जाती है। एक ही wave में दो एजेंट एक ही फ़ाइल में संपादन नहीं कर सकते, इसलिए विलय चरण हमेशा टकराव-मुक्त रहता है।

**worktree पृथक्करण** निष्पादन-समय के हस्तक्षेप को रोकता है। हर एजेंट अपने खुद के git worktree में काम करता है, एक अलग निर्देशिका जो वही git इतिहास साझा करती है पर जिसका स्वतंत्र फ़ाइल-वृक्ष होता है। समवर्ती बिल्ड, टेस्ट और टूल-कैश राइट साझा स्थिति पर टकराते नहीं।

कोई भी बाध्यता दूसरी की जगह नहीं ले सकती। worktree के बिना असंयुक्त स्वामित्व: विलय सुरक्षित है, पर समवर्ती बिल्ड डगमगाते हैं। असंयुक्त स्वामित्व के बिना worktree: निष्पादन स्वच्छ है, पर विलय अनसुलझे टकराव पैदा करता है। दोनों को टिकना ज़रूरी है।

### worktree पृथक्करण रक्षा (6 परतें)

एजेंट हमेशा पृथक्करण निर्देशों का पालन नहीं करते। Polywave worktree पृथक्करण को एक अवसंरचना (infrastructure) समस्या मानता है, न कि सहयोग समस्या, जिसमें hook-आधारित प्रवर्तन (E43) प्राथमिक तंत्र है।

| Layer | Mechanism | Type |
|-------|-----------|------|
| **E43** | **hook-आधारित प्रवर्तन**: Claude Code जीवनचक्र hook (SubagentStart, PreToolUse:Bash, PreToolUse:Write/Edit, SubagentStop) अपने आप पर्यावरण चर इंजेक्ट करते हैं, bash कॉल से पहले cd कमांड जोड़ते हैं, और टूल सीमा पर सीमा-के-बाहर राइट को रोकते हैं। उल्लंघन महज़ पता चलने के बजाय असंभव हो जाते हैं। | **रोकथाम (प्राथमिक)** |
| 0 | **Pre-commit hook**: `polywave-tools create-worktrees` द्वारा अपने आप इंस्टॉल किया जाता है। सक्रिय wave के दौरान main पर कमिट रोकता है। Orchestrator `POLYWAVE_ALLOW_MAIN_COMMIT=1` के ज़रिए बायपास करता है। | रोकथाम |
| 1 | **मैनुअल worktree पूर्व-निर्माण**: Orchestrator किसी एजेंट के शुरू होने से पहले सभी worktree बना देता है | नियतात्मक |
| 2 | **`isolation: "worktree"` पैरामीटर**: हर एजेंट लॉन्च टूल स्तर पर worktree पृथक्करण निर्दिष्ट करता है | टूल-स्तर |
| 3 | **Field 0 स्व-सत्यापन**: एजेंट एक संक्षिप्त जाँच के ज़रिए शाखा की पुष्टि करते हैं (प्राथमिक प्रवर्तन `validate_worktree_isolation` SubagentStart hook है) | सहयोगी |
| 4 | **विलय-समय ट्रिप वायर**: Orchestrator विलय से पहले हर worktree शाखा पर कमिट गिनता है। शून्य कमिट = पृथक्करण विफलता। रिकवरी विकल्पों के साथ रुक जाता है। | नियतात्मक |

## Polywave-Teams (प्रायोगिक)

[`docs/proposals/polywave-teams/`](../../docs/proposals/polywave-teams/) Claude Code Agent Teams का उपयोग करने वाली एक वैकल्पिक निष्पादन परत है। वही प्रोटोकॉल, वही IMPL दस्तावेज़, वही Scout। wave की भीतरी प्लंबिंग अलग है: टीम-साथी पृष्ठभूमि के Agent टूल कॉल की जगह लेते हैं, जो एजेंटों के बीच मैसेजिंग और रीयल-टाइम विचलन चेतावनियाँ प्रदान करते हैं।

## ब्लॉग पोस्ट

इस पैटर्न, इसे स्वयं पर आज़माने से मिले सबक, और प्रोटोकॉल कैसे विकसित हुआ, इस पर चार-भागों की श्रृंखला:

1. [Polywave: A Coordination Pattern for Parallel AI Agents](https://blog.blackwell-systems.com/posts/scout-and-wave/)। पैटर्न: भोले (naive) समानांतरण के विफलता मोड, scout डिलिवरेबल, wave निष्पादन, और brewprune से एक हल किया गया उदाहरण।
2. [Polywave, Part 2: What Dogfooding Taught Us](https://blog.blackwell-systems.com/posts/scout-and-wave-part2/)। ऑडिट-फ़िक्स-ऑडिट लूप, ओवरहेड माप (अनदेखा किए जाने पर 88% धीमा), Quick मोड, और नए प्रोजेक्ट्स के लिए बूटस्ट्रैप समस्या।
3. [Polywave, Part 3: Five Failures, Five Fixes](https://blog.blackwell-systems.com/posts/scout-and-wave-part3/)। skill फ़ाइल 400-लाइन के अखंड (monolith) से कैसे विघटित हुई, संस्करण हेडर क्यों मायने रखते हैं, और वास्तविक विफलताओं से प्रेरित पाँच scout प्रॉम्प्ट फ़िक्स।
4. [Polywave, Part 4: Trust Is Structural](https://blog.blackwell-systems.com/posts/scout-and-wave-part4/)। Scaffold Agent, 5-परत worktree पृथक्करण रक्षा, और शुद्धता सहयोग के बजाय अवसंरचना में क्यों निहित है।

## लाइसेंस

[MIT OR Apache-2.0](../../LICENSE)
