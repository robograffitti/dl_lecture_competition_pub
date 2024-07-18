# DL基礎講座2024最終課題レポート VQA

## はじめに
本講義の最終課題において，Visual Question Answering (VQA) を選択し，README.md の「考えられる工夫の例」を参考に開発を行った．

## 工夫した点（実装済）
### 0. docker コマンド
本課題に取り組むにあたり，VQA-competition ブランチにある Dockerfile でイメージをビルドして開発を行った．Docker を利用した開発に慣れたい意向があったため，Dockerfile が最終課題の中で唯一用意されていたことがVQAを最終課題のテーマとして選択した理由の一つである．

ビルドまでの手順は README.md と同様だが，そのままではローカル環境でGPUが使用不可だったため，以下のように変更した．`--gpus all` を追加し，GPUを利用できるように修正した．また，マルチGPU環境において out of memory が起きた際， `nvidia-smi` でGPUの利用状況を確認し，`--gpus device=1`といったように空いているGPUを指定して利用する等，リソースを意識しながら開発を行った．
```bash
$ docker run -it -d --rm --gpus all --name vqa_hoge -v $PWD:/workspace -w /workspace vqa-0.0.0 bash
```
また，上記において，`-d` オプションを指定することによって，バックグランド実行すれば，ターミナルの新規ウィンドウを立ち上げずに，同一のターミナルからコンテナ内部にアクセスすることができる．
```bash
$ docker exec -it vqa_hoge bash
```
### 1. ベースライン実装
特に問題なく実行できた．スコアは0.42弱で，規定の0.41をやや上回るスコアが出た．

### 2. process_text() の question への適用
スコアは0.41弱で，規定とほぼ同一のスコアとなった．

README.md の"考えられる工夫の例"にある"質問文の前処理"に相当する案で，VQADataSet の __getitem__ の中で one-hot 表現に変換する前に，質問文に適用した．

ベースライン実装からスコアがやや下がり，性能向上が見られなかった．

### 3. BERTによる分散表現
スコアは `0.47833` となり，ベースラインを十分上回る結果となった．

feature/vqa/add-tokenizer のブランチにコードがある．

README.md の"考えられる工夫の例"にある"質問文の表現"に相当する案で，質問文を分散表現に変換する tokenizer を利用することを考えた．One-hot 以外で，words2vec, ELMo, BERT といったものがあるが，今回はBERTを利用した．

### 4. BERTモデルの利用
スコアは`0.47537`となった．

feature/vqa/add-bert のブランチにコードがある．

README.md の"考えられる工夫の例"にある"出力候補の変更"に相当する案で，tokenizer として利用した BERT のモデルを利用して実装することを考えた．

## 工夫した点（実装が間に合わなかった部分）

以下，実装途中で時間切れになったが，工夫として取り入れようとしｔ点について述べる．

### 5. 画像の前処理

feature/vqa/add-bert-with-data-aug のブランチにコードがある（未完成）．

README.md の"考えられる工夫の例"にある"画像の前処理"に相当する案で，第5回の演習にある Data Augmentation を適用することを考えた．

画像データを目視で確認したところ，data augmentation の使用が効果的に見えるデータが数多く確認された．

以下に有用性に関する所感を述べる．

有用性が高いと思われる：
- random rotation -> 角度が傾いている画像が多い
- random cropping -> 見切れている画像が多い
- Gaussian filter -> ブレている画像が多い
- Brightness, Contrast, Saturation -> 明るさがバラバラ 

有用性があると思われるが確信がない：
- Mixup -> 複数の物体が写っている画像が多い
- Random erasing -> オクルージョンがあるようなデータが少ない

性能悪化の可能性があるもの：
- Flipping -> 文字列を含む画像が多い

## おわりに
最終的に，feature/vqa/add-tokenizer ブランチにある BERTによる分散表現の質問文への適用が実装した中では最も高いスコアを出したため，最終提出とした．

質問文に分散表現を適用したことからもわかるように，画像にデータ拡張を施した場合でも，スコアの向上を期待できた可能性があるため，十分な時間を確保して取り組んだつもりではあったが，時間切れで実装が間に合わなかった点が悔やまれる．

最終課題で理解の不足を感じた部分および実装が間に合わなかった点について見直し，過去の課題のlate submissionに改めて取り組むことで，最終的な講義内容の理解を深めたいと改めて実感した．