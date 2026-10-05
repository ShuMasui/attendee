---
name: kosen-report-writer
description: このSKILLは高専のレポートをLaTex形式で書くためのスキルです。ただしPDFへの出力までは行わず.texファイルの作成のみを行います
---

# 高専フォーマットのLaTeXレポート作成スキル
(NITNC-InformationDepartment LaTeX Report Formatting Skill)

このスキルは、奈良高専（NITNC）情報工学科などのレポート用LaTeXフォーマット（`latex_snipets.json`）を適切に使用し、高品質な高専レポート用LaTeXファイルを生成・管理するためのエージェント向け手順・知識ベースです。

---

## 1. ディレクトリ構造とファイルの構成
高専レポートのLaTeXプロジェクトは、基本的に以下の構成をとります。

- **`.tex` ファイル** (メイン原稿。例: `Uwano-05-34-MasuiShu.tex`)
- **`.bib` ファイル** (参考文献データベース。例: `Uwano-05-34-MasuiShu.bib`)
- **`latex_snipets.json`** (エディタ等で活用するスニペット定義ファイル)
- **画像フォルダ** (挿入する図のデータ)
- **`latex-output/`** (ビルド時に生成される中間ファイルやPDFの出力先)

> [!IMPORTANT]
> 既存のソースコードや参考文献ファイルに直接変更を加えないよう、読み取り専用として参考にしてください。

---

## 2. LaTeXドキュメントの基本構造 (プリアンブルと設定)
`latex_snipets.json` 内の `reportTemplate` (prefix: `report`) に基づき、ドキュメントを初期化します。

### ドキュメントクラスと基本パッケージ
- **クラス指定**: `\documentclass[platex,dvipdfmx,12pt]{jsarticle}`
  > [!WARNING]
  > 使用するコンパイラは `platex` です。誤って `uplatex` にしないように注意してください。
- **主要パッケージ**:
  - `graphicx`: 図の挿入用
  - `url` (オプション `[hyphens]`): URLの自動改行付き表示
  - `otf`: 日本語フォント拡張
  - `multirow`: 表内の複数行結合用
  - `amsmath`: 数式環境の拡張
  - `here`: 図表をコード記述位置に強制配置するための `[H]` 指定用
  - `geometry`: 余白調整（左右20mm、上20mm、下30mm）
  - `ascmac`: 枠線環境 (`screen`, `boxnote` など)
  - `caption`, `subcaption` (オプション `[hang,small,bf]`, `[subrefformat=parens]`): 図表のキャプションおよびサブキャプション用
  - `textcomp`: 特殊記号用
  - `listings`: ソースコードの埋め込み用

### ファイル名などをインラインで挿入したい場合
textttなどは使わずに、そのままのテキスト表現をすること

### ソースコード表示の共通設定 (`listings`)
```latex
\usepackage{listings} % 日本語のコメントアウトをする場合jvlisting（もしくはjlisting）が必要
\lstset{
  basicstyle={\ttfamily},              % 基本:等幅フォント
  identifierstyle={\small},            % 識別子
  commentstyle={\smallitshape},        % コメント:斜体
  keywordstyle={\small\bfseries},     % キーワード:太字
  stringstyle={\small\ttfamily},      % 文字列
  frame={tb},                           % 上部と下部に線を表示
  breaklines=true,                      % 行が長い場合に折り返す
  columns=[l]{fullflexible},            % 列幅を自動調整する（見た目が良くなる）
  numbers=left,                         % 行番号を左側に表示
  xrightmargin=0zw,                     % 右マージンのサイズ
  xleftmargin=1.6zw,                    % 左マージンのサイズ（行番号が2桁でもはみ出ない値）
  numberstyle={\scriptsize},           % 行番号のスタイル（スクリプトサイズ）
  stepnumber=1,                         % 何行ごとに行番号を表示するか
  numbersep=1zw,                        % 行番号とソースコードの間の距離
  lineskip=-1.4ex,                      % ソースコードの行間（詰め気味）
  morestring=[b]",                     % 日本語等のStringをダブルクオーテーションで囲む
  showstringspaces=false,               % スペースの記号表示を無効化
}
\renewcommand{\lstlistingname}{ソースコード}
```

### ページ構成および番号付けの設定
- **余白・行間**:
  ```latex
  \geometry{left=20mm,right=20mm,top=20mm,bottom=30mm}
  \renewcommand{\baselinestretch}{1.2}
  ```
- **図表・数式番号のセクション内番号化**:
  図や表、数式の番号を `(章番号).(図番号)` に変更する設定を行います。
  ```latex
  \renewcommand{\theequation}{\arabic{section}.\arabic{equation}}
  \renewcommand{\thefigure}{\arabic{section}.\arabic{figure}}
  \renewcommand{\thetable}{\arabic{section}.\arabic{table}}
  ```
- **セクション見出しの変更**:
  `\section` が実行されたときに `\clearpage` で自動的に改ページし、大見出しを「第 X 章」形式にします。
  ```latex
  \makeatletter
  \renewcommand{\presectionname}{第}
  \renewcommand{\postsectionname}{章} % 注：つけ忘れると\section*でバグります
  \renewcommand{\section}{%
  \clearpage
  \@startsection{section}{1}{\z@}%
  {\Cvs \@plus.5\Cdp \@minus.2\Cdp}% 見出し前に付けられる空白量
  {.5\Cvs \@plus.3\Cdp}% 見出し後に付けられる垂直方向の空白量
  {\normalfont\LARGE\headfont\raggedright}}% 見出しのスタイル
  \makeatother
  ```

### 表紙 (Title Page) とアブストラクト
1. **表紙**: `\begin{titlepage}` 内で中央揃えの巨大なタイトル、右寄せのクラス・番号・氏名、提出期限・提出日、共同実験者一覧（`tabular` 環境を使用）を配置します。
2. **アブストラクト**: `\section*{アブストラクト}` にて `\thispagestyle{empty}` を指定し、ページ番号を含めずにレポートの要約を記述します。
3. **本文開始**: `\section{実験目的}` の直前に `\setcounter{page}{1}` を記述し、ここからページ番号のカウントを開始します。

---

## 3. スニペットを用いた各種要素の挿入方法
`latex_snipets.json` に定義されている各種スニペットパターンに従って各パーツを構築します。

### ① 画像の挿入 (1枚)
スニペット名：`figuretemplate` (prefix: `figure`)
```latex
\begin{figure}[htbp]
  \begin{center}
    \includegraphics[width=.5\textwidth]{画像ファイルパス}
    \caption{画像のキャプション}
    \label{fig:識別キー}
  \end{center}  
\end{figure}
```

### ② 横並び画像の挿入 (2枚まで)
スニペット名：`dobulefiguretemplate` (prefix: `2figure`)
`minipage` 環境と `\subcaption` を用いて、2枚の画像を並列配置します。
```latex
\begin{figure}[htbp]
  \centering
  \begin{minipage}[t]{0.45\textwidth}
    \centering
    \includegraphics[width=.98\textwidth]{画像パス1}
    \subcaption{画像1のサブキャプション}
    \label{fig:sub_label1}
  \end{minipage}\hspace{0.05\textwidth}\begin{minipage}[t]{0.45\textwidth}
    \centering
    \includegraphics[width=.98\textwidth]{画像パス2}
    \subcaption{画像2のサブキャプション}
    \label{fig:sub_label2}
  \end{minipage}
  \caption{全体のキャプション}
\end{figure}
```

### ③ ソースコードエリアの挿入
スニペット名：`code` (prefix: `code`)
`lstlisting` 環境を使用してソースコードブロックを記述します。
```latex
\begin{center}
\begin{lstlisting}[caption={ソースコードの説明/プログラム名}, label={src:識別キー}]
// ここにソースコードを貼り付け
\end{lstlisting}
\end{center}
```

### ④ CLI（コマンドライン出力）の挿入
スニペット名：`cli` (prefix: `cli`)
実行したコマンドやその出力を再現する際、`screen` 環境と `verbatim` 環境を組み合わせて図として挿入します。
```latex
\begin{figure}[H]
\begin{center}
\begin{screen}
\begin{verbatim}
$ command --option argument
[Output messages here]
\end{verbatim}
\end{screen}
\end{center}
\caption{実行コマンドおよび出力結果} 
\label{fig:cli_label}
\end{figure}
```

### ⑤ 表（テーブル）の挿入
スニペット名：`tabletemplate` (prefix: `table`)
```latex
\begin{table}[htbp]
  \centering
  \caption{表の説明}
  \label{tab:識別キー}
  \begin{tabular}{ccc}
    \hline
    ヘッダ1 & ヘッダ2 & ヘッダ3 \\
    \hline
    データ1 & データ2 & データ3 \\
    \hline
  \end{tabular}
\end{table}
```

### ⑥ 特定の演習テンプレート（西野先生のテンプレートなど）
スニペット名：`nishinoTemplate` (prefix: `nishino`)
特定の課題に対する定型セクション構成です。
```latex
\subsection{課題名}
\subsubsection{課題の設問}
\subsubsection{入力したコマンド}
\subsubsection{実験結果}
\subsubsection{結果説明}
```

---

## 4. 参考文献の管理 (BibTeX)
参考文献は `.bib` ファイルで一元管理し、本文中からキーを指定して引用します。

### BibTeXファイルの構成 (例: `Uwano-05-34-MasuiShu.bib`)
Webページなどを引用する場合、`@misc` エントリを用いて以下のように記述します。
```bibtex
@misc{ wiki:aws-cicd-definition,
  author       = {{Amazon Web Services}},
  title        = {CI/CD とは何ですか? - 継続的インテグレーションと継続的デリバリーの説明},
  howpublished = {\url{https://aws.amazon.com/jp/what-is/ci-cd/}},
  year         = {2024},
  note         = "[Online; accessed 2026-01-26]"
}
```
> [!TIP]
> 著者名（`author`）に組織名や日本語の固有名詞を指定する場合、BibTeXの処理による自動省略を防ぐために `{{Amazon Web Services}}` や `"{Wikipedia contributors}"` のように二重中括弧または二重クォーテーションで囲みます。

### 本文中の引用方法
- 本文中では `\cite{参照キー}` を用いて引用します。
  - 例: `テスト関係性をまとめよ\cite{wiki:aws-cicd-definition}`
- 複数の参考文献を同時に引用する場合は、カンマで区切ります。
  - 例: `関係性をまとめよ\cite{wiki:aws-cicd-definition}, \cite{wiki:tdd}`

### 参考文献リストの出力
LaTeX原稿の末尾（`\end{document}` の直前）に以下を記述します。
```latex
\bibliographystyle{jplain}
\bibliography{Uwano-05-34-MasuiShu} % .bib拡張子を除いたファイル名
```

---

## 5. LaTeXレポート作成時のチェックリスト
- [ ] クラス指定が `platex` になっているか（誤って `uplatex` にしていないか）？
- [ ] 各章の見出しには `\section{...}` を使用しているか（`\section` 使用時に自動で `\clearpage` が適用されるため、ページめくりのタイミングに注意する）？
- [ ] 図・表・ソースコード・CLIにはそれぞれ適切なキャプションと `\label` を付与し、本文中では `図\ref{fig:label}` のように動的参照しているか？
- [ ] ソースコードは `code` スニペット（`lstlisting`）、CLI出力は `cli` スニペット（`screen` + `verbatim`）を使い分けているか？
- [ ] 参考文献を引用する箇所に `\cite{...}` が書かれており、ドキュメント末尾に `\bibliography{...}` が指定されているか？
- [ ] BibTeXのビルドプロセス（`platex` -> `pbibtex` -> `platex` -> `platex` -> `dvipdfmx`）が適切に実行できる構造か？
