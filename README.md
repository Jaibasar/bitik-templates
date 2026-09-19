[README.md](https://github.com/user-attachments/files/32409718/README.md)


[setup.sh](https://github.com/user-attachments/files/32409725/setup.sh)
[LICENSE.md](https://github.com/user-attachments/files/32409722/LICENSE.md)
#!/bin/bash
# Бітік шаблон жүйесін орнату: қазақша тасымалдау үлгісін TeX-ке қосу
# Пайдалану: bash setup.sh
#
# ЕСКЕРТУ (2026-09 жаңартуы): қаріптерді жүйеге орнатудың қажеті жоқ —
# bitik-core.tex оларды fonts/ қапшығынан fontspec-тің Path= арқылы
# тікелей жүктейді. Бұл скрипт енді ТЕК қазақша тасымалдау (hyphenation)
# үлгісін тіркейді — ол нақты жүйелік деңгейде (TeX форматына құрастыру
# арқылы) орнатылуы керек, оны Path= сияқты трюкпен айналып өту мүмкін емес.
set -e

echo "1/3 — Қазақша тасымалдау үлгісін жүктеу (hyph-utf8 жобасы)..."
TMPDIR=$(mktemp -d)
curl -sSL -o "$TMPDIR/hyph-kk.tex" \
  "https://raw.githubusercontent.com/hyphenation/tex-hyphen/master/hyph-utf8/tex/generic/hyph-utf8/patterns/tex/hyph-kk.tex"
curl -sSL -o "$TMPDIR/loadhyph-kk.tex" \
  "https://raw.githubusercontent.com/hyphenation/tex-hyphen/master/hyph-utf8/tex/generic/hyph-utf8/loadhyph/loadhyph-kk.tex"

TEXMFLOCAL=$(kpsewhich -var-value TEXMFLOCAL)
mkdir -p "$TEXMFLOCAL/tex/generic/hyph-utf8/patterns/tex"
mkdir -p "$TEXMFLOCAL/tex/generic/hyph-utf8/loadhyph"
cp "$TMPDIR/hyph-kk.tex" "$TEXMFLOCAL/tex/generic/hyph-utf8/patterns/tex/"
cp "$TMPDIR/loadhyph-kk.tex" "$TEXMFLOCAL/tex/generic/hyph-utf8/loadhyph/"
texhash "$TEXMFLOCAL"
rm -rf "$TMPDIR"

echo "2/3 — Тілді language.dat-қа тіркеу..."
LANGDAT="$(kpsewhich language.dat)"
if ! grep -q "^kazakh " "$LANGDAT"; then
  echo "kazakh loadhyph-kk.tex" | sudo tee -a "$LANGDAT" > /dev/null
else
  echo "   (тіркелген, өткізіп жіберілді)"
fi

echo "3/3 — XeLaTeX форматын қайта құрастыру..."
sudo fmtutil-sys --byfmt xelatex

echo ""
echo "Дайын! Тексеру үшін:"
echo "  cd sizes && xelatex five-by-eight.tex"
