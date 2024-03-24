# Code Reading of Tensorflow Lite for Microcontroller

本稿では Tensorflow Lite for Microcontroller のランタイムを理解を目標とする．
方針としては，[Hello world example](https://www.tensorflow.org/lite/microcontrollers/get_started_low_level?hl=ja)を順にたどっていくことで理解する．

## [`g_hello_world_float_model_data` を読み込みモデルのデータを読み込む](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/examples/hello_world/hello_world_test.cc#L70-L72)

`g_hello_world_float_model_data`は`xxd`コマンドで `\*.tflite`から変換された C 配列．
読み込みは`flatc`を使って生成された`schema_generated.h`に定義されている[GetModel](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/examples/hello_world/hello_world_test.cc#L70-L72)を使って読み込む．

## [operator の登録](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/examples/hello_world/hello_world_test.cc#L74-L76)

- [MicroMutableOpResolver](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_mutable_op_resolver.h#L43-L44)クラスの変数を作成して，順次オペレーターを使いしていく．
- [RegisteOps](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/examples/hello_world/hello_world_test.cc#L33-L34)関数の中で全結合層を追加していて，[AddFullyConnected](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_mutable_op_resolver.h#L264-L269)の実装は`micro_mutabel_op_resolver.h`に書かれている．
- [Register_FULLY_CONNECTED()](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/kernels/fully_connected.cc#L202-L205)で全結合層のオペレータを登録している．
  [RegisteOp](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/kernels/kernel_util.cc#L41)で tensorflow kernel の`TFLMRegistration`を返している． - [Init](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/kernels/fully_connected.cc#L29-L34): バッファを allocate する関数．なぜ全結合層専用？． - [Prepare](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/kernels/fully_connected.cc#L35-L67)で入出力のテンソルを確保している． - [Eval](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/kernels/fully_connected.cc#L90-L91)で全結合層の順伝搬用の関数．
- [AddBuiltin](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_mutable_op_resolver.h#L573-L574)で全結合のオペレータを登録している．
  - [BuiltinOperator_FULLY_CONNECTED]: どのオペレータかを判別するための列挙型の要素．
  - `registeration`: `Register_FULLY_CONNECTED()`で返ってきた`TFLMRegistration`構造体．
  - [ParseFullyConnected](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/core/api/flatbuffer_conversions.cc#L1415):おそらく，モデルの構造と重みが記録されている flatbuffer の parse 関数．
  - 返り値：`TfLiteStatus`

## [tensorflow intepretor の初期化](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/examples/hello_world/hello_world_test.cc#L82-L85)

- 各メンバ変数を初期化している．
- [Init](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L79-L94)で初期化状態に更新している．

## [TF_LITE_ENSURE_STATUS(interpreter.AllocateTensors());](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/examples/hello_world/hello_world_test.cc#L84-L85)

- [AllocateTensors](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L188)：ここで，テンソルを allocate している．

- [TfLiteStatus MicroInterpreter::AllocateTensors() ](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L188-L189)：ここで，グラフを構築している．

  - [SubgraphAllocations\* allocations = allocator\*.StartModelAllocation(model\*);](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L189-L190)：model の tensor を allocate している．
    - [uint8_t\* data_allocator_buffer =](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_allocator.cc#L451-L455)：allocate のしているのはここ．
    - [uint8_t\* SingleArenaBufferAllocator::AllocatePersistentBuffer(](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/arena_allocator/single_arena_buffer_allocator.cc#L108-L124)：ここがテンソルの実質的な allocate の場所．
    - [if (AllocateTfLiteEvalTensors(model, output) != kTfLiteOk ||](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_allocator.cc#L472-L474)：ここで，グラフの各ノードのテンソルと順伝搬時に使うテンソルの領域を確保している？
      - [TfLiteStatus MicroAllocator::AllocateNodeAndRegistrations(](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_allocator.cc#L581-L605)：グラフのテンソルとかを確保している模様．
        - [NodeAndRegistration\* output = reinterpret_cast\<NodeAndRegistration\*\>(](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_allocator.cc#L593-L597)：各オペレータのサイズ（各層)の数だけノードを確保している．
      - [TfLiteStatus MicroAllocator::AllocateTfLiteEvalTensors(](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_allocator.cc#L722-L723)：ここで，順伝搬用のテンソルの領域を確保している．
        - [TfLiteStatus status = internal::InitializeTfLiteEvalTensorFromFlatbuffer(](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_allocator.cc#L744-L745)：これが flatbuffer から重みを読み込んで，初期化している．
          - [result->data.data = GetFlatbufferTensorBuffer(flatbuffer_tensor, buffers);](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_allocator.cc#L323-L324):この関数の返り値が void になっており，float32 or int にキャストされる？
        - [TfLiteStatus InitializeTfLiteEvalTensorFromFlatbuffer(](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_allocator.cc#L313-L314):これがテンソルを取得している実態．
        - [GetFlatbufferTensorBuffer](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_allocator.cc#L163)：これが渡された flatbuffer_tensor と buffers の整合性をチェックしている．
  - [SetSubgraphAllocations](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L197)：ここで，subgraph をセットしている．
    - この部分がグラフ構造の初期化をしている．
  - [TF_LITE_ENSURE_STATUS(PrepareNodeAndRegistrationDataFromFlatbuffer());](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L199-L200)：ここが flatbuffer の中身をみて，各 node を準備している？
  - [TfLiteStatus MicroInterpreter::PrepareNodeAndRegistrationDataFromFlatbuffer() ](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L95):flatbuffer の中身を見ているところ．
  - [for (size_t i = 0; i < operators_size; ++i) ](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L105-L106)：ここで，グラフを構築している．

## [graph\*(&context\*, model, &allocator\_, resource_variables),](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L47-L48)ここでグラフ構造の初期値を設定している．

- [TF\*LITE_ENSURE_STATUS(graph\*.InitSubgraphs());](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L202-L203):ここで，サブグラフを構築している．

  - [TfLiteStatus MicroGraph::InitSubgraphs() ](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_graph.cc#L55-L56):
    - [registration->init(context\_, init_data, init_data_size);](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_graph.cc#L79-L80)：ここで，`Init`を読んでいる．
    - [uint32\*t operators_size = NumSubgraphOperators(model\*, subgraph_idx);](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_graph.cc#L61)：ここで，角層の長さを取得している．

- [TF\*LITE_ENSURE_STATUS(graph\*.PrepareSubgraphs());](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L206-L207)：ここで subgraph を準備している．
  - [TfLiteStatus MicroGraph::PrepareSubgraphs() ](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_graph.cc#L88-L89)：これが実装．
  - [TfLiteStatus prepare\*status = registration->prepare(context\*, node);](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_graph.cc#L102-L103)：ここで，各 node の Prepare を呼び出している．

## [TF_LITE_ENSURE_STATUS(interpreter.Invoke());](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/examples/hello_world/hello_world_test.cc#L94-L95)

- [TfLiteStatus MicroInterpreter::Invoke() ](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_interpreter.cc#L268-L281): ここで，順伝搬を実行している．
- [TfLiteStatus invoke\*status = registration->invoke(context\*, node);](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_graph.cc#L194):ここで，`RegisteOp`で登録した Eval 関数を呼び出している．
- [for (size_t i = 0; i < operators_size; ++i) ](https://github.com/kadu-v/tflite-micro-sample/blob/0f674d38fc8becd90fbd943fb7e7c49f808a7019/tensorflow/lite/micro/micro_graph.cc#L177-L178):この for 文で各層をたどって順伝搬を実行している．
- TFLiteNode：DNN のグラフ構造を表現するためのクラス．
