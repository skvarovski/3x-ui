# Skvarovski Fork — Уникальные изменения форка

Этот файл — карта всех изменений, отличающих форк `skvarovski/3x-ui` от upstream `MHSanaei/3x-ui`.

## Backup точки

- **Тег**: `backup-pre-sync-v2.8.13` — состояние до sync с upstream v2.9.x
- **Ветка**: `dev-fork` — копия main с нашими изменениями
- **Патчи**: `.skvarovski-patches/0001-0006*.patch` — экспортированные коммиты для git am

## Уникальные коммиты (поверх upstream 169b216d)

| # | Commit | Что |
|---|---|---|
| 1 | `51caf402` | feat: FinalMask TCP masks (fragment + sudoku) для Xray v26.3.27 |
| 2 | `91e22f2d` | feat: header-custom TCP mask |
| 3 | `3c07835c` | chore: URL fork (skvarovski/3x-ui) в install/update/x-ui/README |
| 4 | `242bc798` | fix: prerelease: false в release.yml |
| 5 | `f34fedfc` | fix: UDP masks (noise, sudoku, header-custom, salamander), xicmp listenIp |
| 6 | `a68072a2` | chore: bump version 2.8.13 |

## Что после sync — НЕ применять (upstream сделал лучше)

- **Xray version v26.3.27** → upstream уже на v26.4.17
- **UDP masks UI** (commit f34fedfc) → upstream полностью переписал в `04b4fb43` и `88dafa6c`:
  - Они добавили xdns с массивами (resolvers/domains)
  - salamander UI для Hysteria2
  - Полная реструктуризация stream_finalmask.html
- **xicmp listenIp** → проверить что upstream сделал

## Что ОБЯЗАТЕЛЬНО применить после sync (наше уникальное)

### A. TCP masks (наша главная фича — upstream НЕ сделал)

Файлы:
- `web/assets/js/model/inbound.js` — класс `TcpMask`, расширить `FinalMaskStreamSettings.tcp[]`, методы `addTcpMask`/`delTcpMask`/`hasFinalMask`, функция `extractFinalMaskParams`, обновить 4 генератора ссылок (vmess/vless/ss/trojan) с `fragment_length`/`fragment_delay`/`sudoku` URL params
- `web/assets/js/model/outbound.js` — то же + URL импорт в `fromParamLink` и `fromVmessLink`
- `web/html/form/stream/stream_finalmask.html` — TCP Masks UI секция (адаптировать к новой структуре от upstream!)
- `sub/subService.go` — `extractFinalMaskParams` Go + 4 генератора (vmess/vless/trojan/shadowsocks)

Поддерживаемые типы:
- `fragment`: packets, length, delay, maxSplit
- `sudoku`: password, ascii, paddingMin, paddingMax
- `header-custom`: clients[][], servers[][], errors[][]

### B. URL fork (заменить mhsanaei/MHSanaei на skvarovski)

Файлы:
- `README.md` — badges, install command
- `install.sh` — все references на MHSanaei/3x-ui
- `update.sh` — все references
- `x-ui.sh` — install_command + другие места

Команда:
```bash
sed -i 's/MHSanaei\/3x-ui/skvarovski\/3x-ui/g; s/mhsanaei\/3x-ui/skvarovski\/3x-ui/g' \
  README.md install.sh update.sh x-ui.sh
```

### C. release.yml prerelease: false

Файл: `.github/workflows/release.yml`
```yaml
prerelease: false  # вместо true (2 места)
```

### D. Версия

Файл: `config/version` — установить новую (2.9.3 если upstream на 2.9.2)

## Правильный workflow на будущее (чтобы не повторилось)

### Стратегия "dev-fork branch"

```
upstream/main  ──●──●──●──●──●──●──  (MHSanaei делает свои релизы)
                  ↓ периодический pull
main           ──●──●──●──●──●──●──  (наша main = mirror upstream)
                  ↓ rebase ↓ rebase
dev-fork       ──●──●──●─────●──●─── (наши уникальные изменения, релизим отсюда)
```

### Конкретные правила

1. **`main` = чистый mirror upstream/main** — никогда не коммитим в main напрямую
2. **`dev-fork` = наша рабочая ветка** — все TCP masks, URL fork, prerelease fix живут здесь
3. **Релизы делаем из `dev-fork`** — теги `v2.8.x-fork` или `v2.8.x-skvarovski`
4. **Sync процесс**:
   ```bash
   git checkout main
   git pull upstream main
   git push origin main
   git checkout dev-fork
   git rebase main          # Переносим наши коммиты поверх свежего upstream
   git push --force-with-lease origin dev-fork
   ```
5. **install.sh URL** должен указывать на `dev-fork` ветку:
   ```
   https://raw.githubusercontent.com/skvarovski/3x-ui/dev-fork/install.sh
   ```

### Альтернатива: ветка `release` для стабильных релизов

```
upstream/main  → main (mirror)
                  ↓
                dev-fork (рабочая)
                  ↓ когда готово
                release (теги ставим тут)
```

## После sync upstream — порядок действий

1. Изучить новый код, особенно `stream_finalmask.html`, `inbound.js`, `outbound.js`, `subService.go`
2. Проверить, что upstream сделал с finalmask — какие типы добавил
3. Применить уникально-наше:
   - **A**: TCP masks (адаптировать UI к новой структуре)
   - **B**: URL fork
   - **C**: prerelease: false
   - **D**: bump версии
4. Проверить релиз на тестовом сервере
5. Зафиксировать workflow `main` ⇄ `dev-fork`
