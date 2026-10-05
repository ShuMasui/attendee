# このコマンドを実行すると以下のようなディレクトリ構造になる

```plaintext
.
├── EngineeringEnglishReport.tex
└── latex-output
    ├── EngineeringEnglishReport.dvi
    ├── EngineeringEnglishReport.pdf
    └── EngineeringEnglishReport.synctex.gz
```

platex -output-directory=latex-output EngineeringEnglishReport.tex && dvipdfmx -o latex-output/EngineeringEnglishReport.pdf latex-output/EngineeringEnglishReport.dvi
