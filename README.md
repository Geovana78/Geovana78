name: Update README (daily quote)

on:
  schedule:
    - cron: "11 5 * * *"   # 05:11 UTC diario
  workflow_dispatch:

jobs:
  refresh:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Generate quote and replace block
        run: |
          set -e
          quotes=(
            "La seguridad bien diseñada se nota porque TODO funciona sin fricción."
            "Menos privilegios, menos riesgo. — Principio de mínimo privilegio"
            "La identidad es el nuevo perímetro. — Zero Trust"
            "Automatiza lo repetible, piensa en lo importante."
            "Primero lo simple, luego lo elegante, después lo inteligente."
          )
          q=${quotes[$RANDOM % ${#quotes[@]}]}

          awk -v RS='' '
            {
              gsub(/<!-- QUOTE-START -->[\\s\\S]*<!-- QUOTE-END -->/,
                   "<!-- QUOTE-START -->\n> " q "\n<!-- QUOTE-END -->")
              print
            }
          ' q="$q" README.md > README.tmp && mv README.tmp README.md

      - name: Commit & push
        uses: stefanzweifel/git-auto-commit-action@v5
        with:
          commit_message: "chore(readme): update daily quote"
          branch: ${{ github.ref_name }}
