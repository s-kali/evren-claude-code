# EVREN LLM → LiteLLM → Claude Code

[EVREN Platformu](https://evren.ssyz.org.tr)'nun OpenAI uyumlu gateway'ini,
Anthropic API bekleyen Claude Code'a bağlayan minimal proxy.

## Kurulum

**1. Anahtar**

```sh
echo "EVREN_API_KEY=evren_llm_..." > .env
```

**2. Kullanım şartları** — bir kez, zorunlu. Yapılmazsa her istek `403` döner.

```sh
source .env

curl -s -H "Authorization: Bearer $EVREN_API_KEY" \
  https://evren-llmapi.ssyz.org.tr/v1/terms/text          # oku

curl -s -X POST -H "Authorization: Bearer $EVREN_API_KEY" \
  -H "Content-Type: application/json" -d '{"version":1}' \
  https://evren-llmapi.ssyz.org.tr/v1/terms/accept        # kabul et
```

**3. Proxy**

```sh
docker compose up -d
```

**4. Claude Code**

```sh
mkdir -p .claude
cp claude-settings.example.json .claude/settings.json
claude
```

> Tüm projelerde geçerli olmasını istiyorsanız `~/.claude/settings.json`
> olarak kopyalayın.

## Model eşlemesi

`claude-settings.json` içinden değiştirilir; `config.yaml`'a dokunmanıza gerek
yok, joker kayıt EVREN'deki tüm modelleri geçirir.

| Claude Code | EVREN | Ne için |
|---|---|---|
| varsayılan | `glm-5.3` | derin akıl yürütme, en yetenekli model |
| Sonnet | `deepseek-v4-flash` | kod ve ajan görevleri, 1M bağlam |
| Haiku | `qwen3.8-flash-next` | arka plan işleri, hızlı |

Oturum içinde `/model` ile geçebilirsiniz.

## Doğrulama

Claude Code araç çağırmaya dayanır; modeller `tools` desteklemezse dosya
okuma/düzenleme döngüsü çalışmaz. Bir kez test edin:

```sh
curl -s http://localhost:4000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"glm-5.3",
       "messages":[{"role":"user","content":"Istanbul hava durumu?"}],
       "tools":[{"type":"function","function":{"name":"get_weather",
         "parameters":{"type":"object","properties":{"city":{"type":"string"}},
         "required":["city"]}}}]}'
```

Yanıtta `tool_calls` varsa yolunda.

## Notlar

- Proxy `127.0.0.1`'e bağlı, dışarıdan erişilemez; bu yüzden ayrı bir proxy
  parolası yok. `ANTHROPIC_AUTH_TOKEN` yalnızca Claude Code boş bırakmadığı
  için var, değeri önemsiz.
- `.env` değişirse: `docker compose up -d --force-recreate`
- Loglar: `docker compose logs -f litellm`
- Joker kayıt sorun çıkarırsa `config.yaml`'da `model_name: "*"` yerine model
  adını (`glm-5.3`) ve `model:` alanına `openai/glm-5.3` yazıp sabitleyin.
- Çıkarım modelleri 2026-11-01'e kadar ücretsiz. Limitler günlük ~10M, dakikalık
  ~500K token; Claude Code büyük bağlam gönderdiği için kota hızlı dolabilir.