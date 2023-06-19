# Code Reading of Tensorflow Lite for Microcontroller
本稿ではTensorflow Lite for Microcontrollerのランタイムを理解を目標とする．
方針としては，[Hello world example](https://www.tensorflow.org/lite/microcontrollers/get_started_low_level?hl=ja)を順にたどっていくことで理解する．

## [`g_hello_world_float_model_data` を読み込みモデルのデータを読み込む](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/examples/hello_world/hello_world_test.cc#L70-L72)
`g_hello_world_float_model_data`は`xxd`コマンドで`*.tflite`から変換されたC配列．
読み込みは`flatc`を使って生成された`schema_generated.h`に定義されている[GetModel](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/examples/hello_world/hello_world_test.cc#L70-L72)を使って読み込む．

## [operatorの登録](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/examples/hello_world/hello_world_test.cc#L74-L76)
- [MicroMutableOpResolver](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_mutable_op_resolver.h#L43-L44)クラスの変数を作成して，順次オペレーターを使いしていく．
- [RegisteOps](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/examples/hello_world/hello_world_test.cc#L33-L34)関数の中で全結合層を追加していて，[AddFullyConnected](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_mutable_op_resolver.h#L264-L269)の実装は`micro_mutabel_op_resolver.h`に書かれている．
- [Register_FULLY_CONNECTED()](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/kernels/fully_connected.cc#L202-L205)で全結合層のオペレータを登録している．  
[RegisteOp](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/kernels/kernel_util.cc#L41)でtensorflow kernelの`TFLMRegistration`を返している．
    - [Init](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/kernels/fully_connected.cc#L29-L34): バッファをallocateする関数．なぜ全結合層専用？．
    - [Prepare](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/kernels/fully_connected.cc#L35-L67)で入出力のテンソルを確保している．
    - [Eval](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/kernels/fully_connected.cc#L90-L91)で全結合層の順伝搬用の関数．
- [AddBuiltin](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_mutable_op_resolver.h#L573-L574)で全結合のオペレータを登録している．
    - [BuiltinOperator_FULLY_CONNECTED]: どのオペレータかを判別するための列挙型の要素．
    - `registeration`: `Register_FULLY_CONNECTED()`で返ってきた`TFLMRegistration`構造体．
    - [ParseFullyConnected](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/core/api/flatbuffer_conversions.cc#L1415):おそらく，モデルの構造と重みが記録されているflatbufferのparse関数．
    - 返り値：`TfLiteStatus`


## [tensorflow intepretorの初期化](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/examples/hello_world/hello_world_test.cc#L82-L85)
- 各メンバ変数を初期化している．
- [Init](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L79-L94)で初期化状態に更新している．


## [TF_LITE_ENSURE_STATUS(interpreter.AllocateTensors());](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/examples/hello_world/hello_world_test.cc#L84-L85)

- [AllocateTensors](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L188)：ここで，テンソルをallocateしている．
- [TfLiteStatus MicroInterpreter::AllocateTensors() {](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L188-L189)：ここで，グラフを構築している．
    - [SetSubgraphAllocations](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L197)：ここで，subgraphをセットしている．
    - [TF_LITE_ENSURE_STATUS(PrepareNodeAndRegistrationDataFromFlatbuffer());](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L199-L200)：ここがflatbufferの中身をみて，各nodeを準備している？
    - [TfLiteStatus MicroInterpreter::PrepareNodeAndRegistrationDataFromFlatbuffer() {](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L95):flatbufferの中身を見ているところ．
    - [for (size_t i = 0; i < operators_size; ++i) {](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L105-L106)：ここで，グラフを構築している．

    - [graph_(&context_, model, &allocator_, resource_variables),](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L47-L48)ここでグラフ構造の初期値を設定している．
        - 
    - [TF_LITE_ENSURE_STATUS(graph_.InitSubgraphs());](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L202-L203):ここで，サブグラフを構築している．
        - [TfLiteStatus MicroGraph::InitSubgraphs() {](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_graph.cc#L55-L56):
            - [registration->init(context_, init_data, init_data_size);](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_graph.cc#L79-L80)：ここで，`Init`を読んでいる．
            - [uint32_t operators_size = NumSubgraphOperators(model_, subgraph_idx);](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_graph.cc#L61)：ここで，角層の長さを取得している．

    - [TF_LITE_ENSURE_STATUS(graph_.PrepareSubgraphs());](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L206-L207)：ここでsubgraphを準備している．
        - [TfLiteStatus MicroGraph::PrepareSubgraphs() {](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_graph.cc#L88-L89)：これが実装．
        - [TfLiteStatus prepare_status = registration->prepare(context_, node);](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_graph.cc#L102-L103)：ここで，各nodeのPrepareを呼び出している．
## [TF_LITE_ENSURE_STATUS(interpreter.Invoke());](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/examples/hello_world/hello_world_test.cc#L94-L95)
- [TfLiteStatus MicroInterpreter::Invoke() {](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L268-L281): ここで，順伝搬を実行している．
- [TfLiteStatus invoke_status = registration->invoke(context_, node);](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_graph.cc#L194):ここで，`RegisteOp`で登録したEval関数を呼び出している．
- [for (size_t i = 0; i < operators_size; ++i) {](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_graph.cc#L177-L178):このfor文で各層をたどって順伝搬を実行している．
- TFLiteNode：DNNのグラフ構造を表現するためのクラス．