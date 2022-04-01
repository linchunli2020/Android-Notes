<img width="525" alt="image" src="https://user-images.githubusercontent.com/67937122/161221282-53f33659-ab12-4237-9542-508a2eaaa61d.png">

（一）Activity A 启动 Activity B，回调如下：

    Activity A的onPause()-> Activity B 的onCreate() -> onStart() -> onResume() -> Activity A的onStop();

    如果B是透明主题，又或者是个DialogActivity，则不会回调A的onStop()。

（二）使用onSaveInstanceState()保存简单，轻量级的UI状态

          lateinit var textView: TextView
          var gameState: String? = null

          override fun onCreate(savedInstanceState: Bundle?) {
              super.onCreate(savedInstanceState)
              gameState = savedInstanceState?.getString(GAME_STATE_KEY)
              setContentView(R.layout.activity_main)
              textView = findViewById(R.id.text_view)
          }

          override fun onRestoreInstanceState(savedInstanceState: Bundle?) {
              textView.text = savedInstanceState?.getString(TEXT_VIEW_KEY)
          }

          override fun onSaveInstanceState(outState: Bundle?) {
              outState?.run {
                  putString(GAME_STATE_KEY, gameState)
                  putString(TEXT_VIEW_KEY, textView.text.toString())
              }
              super.onSaveInstanceState(outState)
          }
