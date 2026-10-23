# Итоговая работа по Git — пошаговые команды (Git Bash)

Все команды проверены на копии вашего проекта. Замените:
- `USERNAME` — ваш логин GitHub

Если Git просит ввести сообщение в редакторе — сохраните и закройте его (или используйте `-m`).

---
## 2. Подготовка
```bash
cd git_final_project
git branch --show-current ; git status ; git log --oneline --all --graph --decorate
```
На GitHub: New repository → имя `final-git-control`, **без** README/.gitignore/license.
```bash
git remote add origin https://github.com/USERNAME/final-git-control.git
git push -u origin main
git push origin ready-change
```

## 3.1 .gitignore
```bash
printf '*.log\n*.tmp\n*.env\n' > .gitignore
git check-ignore -v temp/debug.log          # покажет правило *.log
git status                                    # debug.log, local.tmp, local.env не видны
git add .gitignore
git commit -m "Добавлен .gitignore"
```

## 3.2 Рабочая область, индекс, amend
```bash
echo "Итоговая самостоятельная работа выполняется студентом" >> README.md
echo "Проверить итоговую версию на GitHub" >> docs/tasks.txt
git add README.md
git diff               # не подготовлено: docs/tasks.txt
git diff --staged      # подготовлено: README.md
git add docs/tasks.txt
git commit -m "Обновлены README и список задач"
echo "Пермяков Ярослав Владимирович, группа 09.07.34" >> README.md
git add README.md
git commit --amend --no-edit
git log --oneline -3 ; git show --stat HEAD     # 1 коммит, 2 файла
```

## 3.3 restore / stash
```bash
sed -i 's/^STATUS=.*/STATUS=temp/' src/app.txt
git diff
git restore src/app.txt                           # отмена без коммита

sed -i 's/^STATUS=.*/STATUS=temp2/' src/app.txt
git add src/app.txt
git restore --staged src/app.txt                  # убрали из индекса, правка осталась
git status
git restore src/app.txt                           # вернуть файл как был

echo "Незавершённая работа" >> notes/team.txt
git stash push -m "wip"
git status                                        # чисто
git stash list                                    # <-- СКРИНШОТ (до восстановления)
git stash pop
git stash list                                    # пусто
sed -i 's/^Незавершённая работа$/Незавершённая работа восстановлена/' notes/team.txt
git commit -am "team: незавершённая работа восстановлена"
```

## 3.4 feature-summary: rebase и merge
```bash
git switch -c feature-summary main
printf 'Final Git Project\nПермяков Ярослав Владимирович\nгруппа 09.07.34\n' > docs/summary.txt
git add docs/summary.txt && git commit -m "Добавлен summary"
echo "Статус: готовится итоговая версия" >> docs/summary.txt
git commit -am "Обновлён статус summary"

git switch main
echo "Основная ветка обновлена перед объединением" >> README.md
git commit -am "README: основная ветка обновлена"

git switch feature-summary
git rebase main
git switch main
git merge feature-summary                         # (можно --no-ff, если хотите видеть коммит слияния)
git log --oneline --graph -6
```

## 3.5 Конфликт
```bash
git switch -c conflict-practice
sed -i 's/^Текущая версия:.*/Текущая версия: версия из conflict-practice/' README.md
git commit -am "conflict-practice: версия"

git switch main
sed -i 's/^Текущая версия:.*/Текущая версия: итоговая/' README.md
git commit -am "main: итоговая версия"

git merge conflict-practice                       # CONFLICT
git status                                        # <-- СКРИНШОТ (конфликт до разрешения)
```
Откройте `README.md`, удалите маркеры `<<<<<<<`, `=======`, `>>>>>>>` и обе строки версии, оставьте одну:
`Текущая версия: итоговая 1.0`
```bash
git add README.md
git commit -m "Merge conflict-practice: конфликт разрешён"
git status                                        # чисто
grep -n '<<<<\|>>>>' README.md                    # пусто
```

## 3.6 cherry-pick
```bash
git log --oneline ready-change                    # найти «Добавлен шаблон итогового отчёта» (edc2db6)
git cherry-pick edc2db6
git show --stat HEAD                              # только docs/report-template.txt
git log -1 --format=%h                            # <-- ХЕШ для отчёта (новый хеш в main)
ls notes                                          # experimental-note.txt нет
git push origin main
```

## 3.7 reset / revert / reflog
```bash
echo "Первая версия" > notes/reset-test.txt
git add notes/reset-test.txt && git commit -m "reset-test: первая версия"
git reset --soft HEAD~1                           # изменение осталось в индексе
echo "Исправленная версия после reset" > notes/reset-test.txt
git add notes/reset-test.txt
git commit -m "reset-test: исправленная версия"

echo "Временная опубликованная строка" >> notes/team.txt
git commit -am "team: временная опубликованная строка"
git push origin main
git revert --no-edit HEAD
git push origin main
git reflog -8                                     # <-- 1–3 строки в отчёт
```
Пояснение для отчёта (пример): `reset: moving to HEAD~1` — отменил последний локальный коммит, оставив изменения в индексе; `commit: ...` — создал исправленный коммит; `revert: ...` — отменил опубликованный коммит новым, не переписывая историю.

## 3.8 Fetch
1. GitHub → `README.md` → ✏️ Edit → добавьте строку `Изменение выполнено через GitHub` → Commit changes (в main).
```bash
git fetch origin
git log --oneline main..origin/main               # коммит с сайта
git pull                                          # или git merge origin/main
git status -sb ; grep GitHub README.md
```

## 3.9 blame и bisect
```bash
git blame -L '/TAX_RATE/,+1' src/app.txt          # короткий хеш → da2fd0c

git bisect start
git bisect bad HEAD
git bisect good $(git rev-list --max-parents=0 HEAD)    # первый коммит
# на каждом шаге:
grep TAX_RATE src/app.txt                         # TAX_RATE=20 → git bisect good ; TAX_RATE=200 → git bisect bad
# ... до сообщения "is the first bad commit"
git bisect reset
git branch --show-current                         # main
```
Ожидаемый результат (по ключу и моей проверке): **da2fd0c — «Обновлены параметры расчёта»** (~4 шага).

## 3.10 Финал
```bash
git branch --show-current ; git status
git branch -a
git log --oneline --graph --all --decorate        # <-- СКРИНШОТ
git tag -a final-1.0 -m "Итоговая самостоятельная работа"
git push origin final-1.0
git tag
git push origin --all
git push origin --tags
```
Ветки `feature-summary` и `conflict-practice` попадут на GitHub через `--all`.

---
## Шаблон отчёта
- ФИО, группа: Пермяков Ярослав Владимирович, 09.07.34
- Ссылка: https://github.com/USERNAME/final-git-control
- Хеш cherry-pick: ______ (исходный edc2db6)
- blame TAX_RATE: da2fd0c
- bisect: da2fd0c — «Обновлены параметры расчёта»
- reflog: (1–3 строки + пояснение)
- Скриншоты: git status (финал), git log --graph, ветки (`git branch -a`), `git tag`/тег на GitHub, `git stash list`, конфликт.
