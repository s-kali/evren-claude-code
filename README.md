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

Hangi modelin hangi rolde kullanılacağı `claude-settings.json` içinden
değiştirilir. Modellerin kendisi `config.yaml`'da tanımlıdır.

| Claude Code | EVREN | Ne için |
|---|---|---|
| varsayılan / Sonnet | `deepseek-v4-flash` | kod ve ajan görevleri, 1M bağlam, hızlı |
| Haiku | `qwen3.8-flash-next` | arka plan işleri |
| Opus | `glm-5.3` | derin akıl yürütme — yavaş, bilerek seçin |

Oturum içinde `/model` ile geçebilirsiniz.

> - `glm-5.3` filonun en yetenekli modeli ama kuyrukta uzun bekleyebiliyor
> (ölçülen: tek istekte ~3 dakika). Claude Code tur başına birçok istek attığı
> için varsayılan `deepseek-v4-flash`. Derin düşünme gerektiren tek bir soru
> için `/model opus` ile geçin.
>
> - `/v1/models` yalnızca `config.yaml`'da tanımlı modelleri listeler. Başka bir
>  EVREN modeli kullanmak için oraya aynı kalıpta bir kayıt ekleyin.

## Notlar

- Çıkarım modelleri 2026-11-01'e kadar ücretsiz. Limitler günlük ~10M, dakikalık
  ~500K token; Claude Code büyük bağlam gönderdiği için kota hızlı dolabilir.