# Dokumentacja usługi Llamafile

Usługa uruchamia gotowy model językowy (llamafile) jako demona `systemd`.  
Działa pod osobnym użytkownikiem systemowym i udostępnia REST API wyłącznie na adresie host-only.

---

## 1. Wdrożenie krok po kroku

### 1.1 Utworzenie użytkownika systemowego

```bash
sudo useradd -r -s /bin/false llamafile
```

### 1.2 Przygotowanie pliku i uprawnień

```bash
sudo mkdir -p /opt/llamafile
sudo mv /sciezka/do/twojego-modelu.llamafile /opt/llamafile/model.llamafile
sudo chmod +x /opt/llamafile/model.llamafile
sudo chown -R llamafile:llamafile /opt/llamafile
```

### 1.3 Jednostka systemd

Utwórz plik `/etc/systemd/system/llamafile.service` o następującej treści:

```ini
[Unit]
Description=Llamafile Service
After=network-online.target
Wants=network-online.target

[Service]
User=llamafile
Group=llamafile
WorkingDirectory=/opt/llamafile
Environment=HOME=/opt/llamafile
ExecStart=/bin/sh -c 'exec /opt/llamafile/model.llamafile --server --host 192.168.100.138 --port 8080'
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

### 1.4 Uruchomienie i autostart

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now llamafile.service
```

---

## 2. Weryfikacja działania

### 2.1 Nasłuchiwanie portu (`ss -tlnp`)

Usługa nasłuchuje wyłącznie na adresie host-only:

```text
State       Recv-Q Send-Q Local Address:Port               Peer Address:Port
LISTEN      0      128    192.168.56.101:8080              0.0.0.0:*           users:(("model.llamafile",pid=1420,fd=3))
```

### 2.2 Przykładowe zapytanie REST API

Zapytanie z gospodarza:

```bash
curl -s http://192.168.100.138:8080/v1/chat/completions -H "Content-Type: application/json" -d '{"model": "LLaMA", "messages": [{"role": "user", "content": "czesc jak sie masz"}]}'
```

Odpowiedź modelu:

```json
{"choices":[{"finish_reason":"stop","index":0,"message":{"role":"assistant","content":"Cześć! Jak się masz? Jestem w stanie pomóc Ci. Co dzisiaj może potrzebowaćś zapytać?"}}],"created":1793632229,"model":"/zip/Qwen3.5-0.8B-Q8_0.gguf","system_fingerprint":"b1789468725-0177dcc73","object":"chat.completion","usage":{"completion_tokens":26,"prompt_tokens":17,"total_tokens":43,"prompt_tokens_details":{"cached_tokens":0}},"id":"chatcmpl-9F8NO9d4wL7FhspHmfoDJHGjhSjYNbfX","timings":{"cache_n":0,"prompt_n":17,"prompt_ms":301.599,"prompt_per_token_ms":17.74111764705882,"prompt_per_second":56.36623463605648,"predicted_n":26,"predicted_ms":1273.154,"predicted_per_token_ms":50.92616,"predicted_per_second":19.636273380910716}}
```

### 2.3 Zużycie pamięci

```text
● llamafile.service - Llamafile Service
     Loaded: loaded (/etc/systemd/system/llamafile.service; enabled; preset: disabled)
     Active: active (running) since Mon 2026-11-02 16:07:43 CET; 5min ago
 Invocation: cccf767c0658426da9b0bab6b5632801
   Main PID: 5298 (.ape-1.10)
      Tasks: 23 (limit: 11876)
     Memory: 844.5M (peak: 845.6M)
        CPU: 33.544s
     CGroup: /system.slice/llamafile.service
             └─5298 /opt/llamafile/.ape-1.10 /opt/llamafile/model.llamafile --server --host 192.168.100.138 ->

```
