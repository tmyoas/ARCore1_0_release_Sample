# ARCore 1.0 触ってみた

## ARCoreとは
Googleが作っているAndroid向けARプラットフォーム
AppleになるとARKit

## preview版からの変更点
- 対応端末が増えた
- 床面しか検出できなかったのが壁面も検出できるようになった(らしい)

## Sampleを動かす
- HelloAR: 平面認識 + オブジェクトを任意の平面に配置、定位
  - 画像から頂点を認識、加速度+地磁気センサで地面に対して平行かどうかを判定している?
  - 平面でなくても頂点にオブジェクトを配置できる

- ComputerVision
  - リアルタイムでエッジ抽出を行う
  - 物体検出のための抽出なので、デフォルトでは閾値(変えられる?)が高め。グラデーションとかには反応しない。

### 環境

- 端末: Zenfone AR (ZS571KL-BK128S8)
  - CPU: Snapdragon 821
  - Mem: 8GB
  - Depthセンサ搭載
    - ARCoreでは使わないのでただの飾り
- 開発環境: Unity2017.3.0f3 Personal 64bit
  - Unityを触ってみたかった / 平行してWindows MR Headsetも買ってた

### やったこと
GooglePlayからARCoreを端末にDL
1を参考にSample(HelloAR)をUnityで展開
-> 動かない。 "arcore encountered a problem connecting. please start the app again."
-> エラーの発生個所を見ると、ARCoreのsession lifecycle上アプリ起動時に即死亡
-> サンプルアプリの設定が間違っていないことを確認したうえで、ARCoreを再インストール
-> うごいた

物理演算を適用してみる
目標: andyを落として検出した平面の上に乗せる
2,4を参考に
- TrackedPlaneVisualizer.prefabにMesh Colider追加
- Andy.prefabにRigidbody追加
- TrackedPlaneVisualizer.csの＿UpdateMeshIfNeeded()を編集
- HelloARController.csのUpdate()を編集
-> Andyが飛ぶようになった、けど平面で拾えてないっぽい

6を参考に
- collider (Capsule Collider) をAndyに追加
-> 挙動が謎。Colliderの大きさY方向にでかい&dotに反応している?
-> TrackedPlaneVisualizerのTransformの値を変えてみる -> あんま変わらない
-> いろいろやってみると、andyの物理演算の値が共有されてるっぽいのと、mesh Colliderの当たり判定が想定以上にY軸方向にプラスにあるのが問題っぽい
-> というか見えてないけどColliderがでかい、のでmesh colliderつけてみる -> 反応しない
-> つーかmesh colliderめっちゃ重い
-> 8を見るに、mesh colliderは動くものに設定するものではないらしい `オブジェクトを動かしている場合（車など）、メッシュコライダーは使用できません。代わりに、プリミティブコライダーを使用する必要があります。この場合、 Generate Colliders 設定を無効にしてください。`
-> primitive collider (cube)をつけて、目測でサイズを設定
-> primitiveに円柱はない & capsuleはheightとradiusしかない(radius=0だと線になる?)

### 次にやりたいこと
- 平面の面積測定して、オブジェクトサイズより大きいとき置かない(警告のポップアップを出す)
- テストの実行(参考: 9)

### 参考
- 1: https://developers.google.com/ar/develop/unity/quickstart
- 2: https://qiita.com/taptappun/items/a5337d29a43d5d673c7f
- 3: https://github.com/google-ar/arcore-unity-sdk
- 4: http://tks-yoshinaga.hatenablog.com/entry/2017/09/19/170800
- 5: http://yuhintosh.hateblo.jp/entry/2018/03/14/014205
- 6: https://qiita.com/yando/items/0cd2daaf1314c0674bbe
- 7: https://developers.google.com/ar/develop/unity/guides/hello-ar-sample
- 8: https://docs.unity3d.com/jp/current/Manual/class-Mesh.html 
- 9: https://www.slideshare.net/hirotoimoto1/unityunittest

ARKitができること

    ポジショントラッキング
    平面検出
    カメラ画像の取得
    現実空間とのスケール一致
    周囲の明るさ推定
    ポイントクラウド（3次元の点座標情報）取得
    現実空間への当たり判定（HitTest）

ARKitができないこと

    マーカートラッキング
    取得した情報の保存/読み込み
    現実空間の位置との関連付け
    ポイントクラウドからのジオメトリ生成
    精度の高いリアルタイムなポイントクラウド取得


### Git
github管理のためにしたこと
- 1: http://tech.guitarrapc.com/entry/2017/07/14/031046#Unity-%E3%81%B8%E3%81%AE%E3%83%91%E3%83%83%E3%82%B1%E3%83%BC%E3%82%B8%E5%B0%8E%E5%85%A5

### あとで読む
https://unitylist.com/p/9z4/ar-core-utils

