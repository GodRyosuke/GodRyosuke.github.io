# fastaiとMNISTデータセットを使って学習させる
PytorchとfastaiではじめるDeepLearningという本を読んでいます。
そこに、MNISTのすべての数字を学習させる課題がありました。

巷に出回っているMNISTを使った学習方法には、fastaiライブラリを使ったものが見当たらなかったので、
自分で考えてつくってみました。


## 使用環境
Google Colaboratory

## 処理の流れ
ぼく個人の理解では、
1.  DataBlock作成
2.  DataLoaders(dls)作成
3.  Learner作成
4.  とりあえず少ないepochで学習させて、学習結果のベースラインを求める
5.  学習率などの外部パラメータの最適化をする

が基本の流れなのかなって思っています！


## fastaiに関するデータをpip install
冒頭の!pip install fastai --upgradeを忘れずに！
```python
!pip install fastai --upgrade
from fastai.vision.all import *
path = untar_data(URLs.MNIST)
```

## データセットの中身を見てみる
```python
path.ls() # ディレクトリの構造を一覧表示

# pathから画像ファイルを勝手に再帰的に読んでくれる
img_files = get_image_files(path)
img_files

# 試しに表示
img = PILImage.create(img_files[0])
img.shape, img.show()
```

## ラベルを生成する関数
学習には画像と、それが何を表しているのかを示すラベル(従属変数)が必要！
今は、画像が入っているディレクトリがラベルの役割を果たしてくれるよ!
```python
# ラベル生成関数
# param r: 画像ファイルのpath
def get_y(r):
  return r.parent.name

get_y(img_files[0])
```

## fastaiの要のDataBlockを作成

```python
mnist = DataBlock(
    blocks=(ImageBlock, CategoryBlock),
    get_items=get_image_files,
    get_y=get_y,
    splitter=FuncSplitter(lambda o: o.parent.parent.name=='testing'), # testingフォルダに入っている画像は検証用データセット
    batch_tfms=[Normalize.from_stats(*imagenet_stats)]  #正規化
)

dls = mnist.dataloaders(path)
dsets = mnist.datasets(path)
```

## Learner作成
```python
learn = cnn_learner(dls, resnet18, metrics=error_rate)
learn.fine_tune(2, base_lr=2e-4)
```

## 学習率最適化
fastaiの学習率ファインダで最適な学習率を探す！
```python
lr_min = learn.lr_find()
lr_min
```
それで見つかった学習率が僕の環境では0.00015848931798245758だったので、それを指定してやる。
まず、転移学習させる際に、ほかの層のパラメータは更新させずに(freeze)、追加された最終層のみ
学習実行。
```python
learn.fit_one_cycle(3, 0.00015848931798245758)
```
そんで、個別学習率で学習。
フリーズ解除すると、最適な学習率も変わるので、再度lr_findする。
```python
learn.unfreeze()  # フリーズ解除
lr_min = learn.lr_find()   # 新しくなったモデルの最適な学習率を探す
lr_min
```
学習しておわり。
```python
learn.fit_one_cycle(12, lr_max=slice(1e-6, 1e-4))
```




