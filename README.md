# Xdebug + Laravel no VS Code (Step Debug)

Guia para configurar **step debugging** (breakpoints, step over/into, inspeção de variáveis) em um projeto **Laravel** usando **VS Code**.

## Pré-requisitos

- PHP instalado (`php -v`)
- Um projeto Laravel
- VS Code

---

## 1) Instalar o Xdebug

A instalação muda conforme o sistema. Siga **uma** das opções da doc oficial:

- Via **package manager** (ex.: `apt`, `yum`, etc.)
- Via **PECL**
- Via **PIE**
- Via **source**

Fonte: Xdebug — Installation: https://xdebug.org/docs/install

---

## 2) Habilitar e configurar no `php.ini`

1. Descubra qual `php.ini` está sendo usado:
   ```bash
   php --ini
   ```

2. Abra o `php.ini` (ou o arquivo `.ini` carregado) e adicione/ajuste:

```ini
; ===== Xdebug 3 — Step Debug =====
zend_extension=xdebug

xdebug.mode=debug
xdebug.start_with_request=trigger

; Xdebug 3 usa 9003 por padrão
xdebug.client_port=9003

; IDE e PHP na mesma máquina
xdebug.client_host=127.0.0.1

; opcional: log para troubleshooting (ligue só enquanto investiga)
; xdebug.log=/tmp/xdebug.log
; xdebug.log_level=7
```

Notas importantes (doc oficial):

- `xdebug.mode=debug` habilita o step debugger.
- `xdebug.start_with_request=trigger` inicia o debug **somente** quando houver gatilho.
- Em modo `trigger`, o gatilho é `XDEBUG_TRIGGER` via env/GET/POST/COOKIE.
- Porta padrão do Xdebug 3 para step debug: **9003**.

Fontes:
- Xdebug — Step Debugging: https://xdebug.org/docs/step_debug  
- Xdebug — All settings: https://xdebug.org/docs/all_settings

3. Reinicie o processo do PHP:
   - Se estiver usando `php artisan serve`, finalize e execute de novo.
   - Se estiver usando PHP-FPM/Apache/Nginx, reinicie o serviço correspondente.

---

## 3) Validar que o Xdebug carregou

```bash
php -v
php --ri xdebug
```

Você deve ver o Xdebug listado e as diretivas carregadas.

---

## 4) Configurar o VS Code

### 4.1 Instalar a extensão

Instale a extensão oficial:

- **PHP Debug** (repositório: https://github.com/xdebug/vscode-php-debug)

### 4.2 Configurar `.vscode/launch.json`

Crie/edite o arquivo `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Listen for Xdebug",
      "type": "php",
      "request": "launch",
      "port": 9003
    }
  ]
}
```

Fonte: README do vscode-php-debug: https://github.com/xdebug/vscode-php-debug/blob/main/README.md

### 4.3 Iniciar o listener

1. Vá em **Run and Debug**
2. Selecione **Listen for Xdebug**
3. Clique em **Start Debugging**

---

## 5) Iniciar uma sessão de debug (gatilho)

Como você configurou `xdebug.start_with_request=trigger`, o Xdebug só vai conectar quando você “pedir”.

### 5.1 HTTP (browser / API)

Opções comuns:

- Adicionar na URL:
  - `?XDEBUG_TRIGGER=1`
- Enviar header `XDEBUG_TRIGGER: 1` (Postman/Insomnia/curl)
- Enviar cookie `XDEBUG_TRIGGER=1`

Fonte: Xdebug — All settings (`xdebug.start_with_request` / trigger): https://xdebug.org/docs/all_settings

### 5.2 CLI (Artisan)

Exemplos:

```bash
XDEBUG_TRIGGER=1 php artisan test
XDEBUG_TRIGGER=1 php artisan tinker
XDEBUG_TRIGGER=1 php artisan queue:work
```

---

## 6) Onde colocar breakpoints no Laravel (atalhos)

- `routes/web.php` e `routes/api.php` (teste rápido)
- Controllers: `app/Http/Controllers`
- Middlewares: `app/Http/Middleware`
- Jobs/Queues: `app/Jobs`
- Commands: `app/Console/Commands`

---

## Troubleshooting (checklist rápido)

### A) Xdebug não aparece no `php --ri xdebug`
- Você editou o `php.ini` errado → confirme com `php --ini`.
- Reinicie o processo do PHP após mudar config.

### B) “Could not connect to debugging client”
- VS Code está em **Listen for Xdebug** (rodando)?
- Porta bate nos dois lados: `xdebug.client_port=9003` e `"port": 9003`.
- Firewall bloqueando 9003?

Fonte (porta padrão e comportamento): https://xdebug.org/docs/all_settings

### C) Breakpoints não param
- O request está indo com `XDEBUG_TRIGGER`?
- O código que você abriu no VS Code é o mesmo diretório que o PHP está executando?
- Se você tiver múltiplos PHPs (CLI vs servidor), confirme qual ini cada um usa.

---

## Referências oficiais

- Xdebug — Installation: https://xdebug.org/docs/install  
- Xdebug — Step Debugging: https://xdebug.org/docs/step_debug  
- Xdebug — All settings: https://xdebug.org/docs/all_settings  
- VS Code PHP Debug (xdebug/vscode-php-debug): https://github.com/xdebug/vscode-php-debug
