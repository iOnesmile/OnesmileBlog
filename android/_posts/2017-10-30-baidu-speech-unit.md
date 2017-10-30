---
layout: post
title: "百度语音和 UNIT 使用笔记"
modified: 2017-10-30 12:48:00
excerpt: "语音识别、语音唤醒、语音合成、UNIT..."
tags: [Speech]
comments: true
---



文档：<http://ai.baidu.com/docs#/Begin/top>

SDK 下载：<http://ai.baidu.com/sdk#asr>

#### 1. 语音识别

- 在线识别见文档，略。   
- 离线识别
	- 目前没有公开版本的离线任意语句识别
	- 只能识别预定义的短语，在 <http:/yuyin.baidu.com/asr> 页面下方可以自行定义 bsg 文件
	- 在代码中可以动态定义“词条”

	```
	// MyRecognizer 来自 Demo 中
	myRecognizer = new MyRecognizer(this, recogStateListener);
	Map<String, Object> map = new HashMap<String, Object>();
	map.put(SpeechConstant.DECODER, 2);
	// 定义离线语音包文件
	map.put(SpeechConstant.ASR_OFFLINE_ENGINE_GRAMMER_FILE_PATH, "asset:///baidu_speech_grammar.bsg");
	map.putAll(fetchSlotDataParam());
	myRecognizer.loadOfflineEngine(map);
	
	// 动态修改“词条”，其中 "turnOn" 和 "exit" 键为 bsg 语音包中定义的
	public Map<String, Object> fetchSlotDataParam(){
	    Map<String, Object> map = new HashMap<String, Object>();
	    try {
	        JSONObject json = new JSONObject();
	        json.put("turnOn", new JSONArray().put("开灯").put("芝麻开灯"))
	                .put("exit", new JSONArray().put("退出").put("再见").put("滚").put("关闭页面"));
	        map.put(SpeechConstant.SLOT_DATA, json);
	    } catch (JSONException e) {
	        e.printStackTrace();
	    }
	    return map;
	}
	
	private void startRecognizer() {
	    Map<String, Object> params = new HashMap<>();
	    // 表示启用离线功能，但是SDK强制在线优先
	    params.put(SpeechConstant.DECODER, 2);
	    ...
	    myRecognizer.start(params);
	}
	```
	
	
#### 2. 语音唤醒

唤醒词评估和生成： <http://ai.baidu.com/tech/speech/wake>


#### 3. 语音合成

在线测试试音效果： <http://speech.baidu.com/#try>

- 离线合成
	- 只支持标准男声和女生
	- 耗时较短但效果不如在线好
	- 需要认证
	- 对于发音响应有比较高要求，可以使用 `PARAM_MIX_MODE` 来设置模式

	```
    private void initTTs(Context context) {
        mSpeechSynthesizer = SpeechSynthesizer.getInstance();
        mSpeechSynthesizer.setContext(context);
        mSpeechSynthesizer.setSpeechSynthesizerListener(this);

        mSpeechSynthesizer.setAppId(BAIDU_APP_ID);
        mSpeechSynthesizer.setApiKey(BAIDU_APP_KEY, BAIDU_APP_SECRET);

        // 初始化离线合成参数
        initOfflineResource();

        // 以下setParam 参数选填。不填写则默认值生效
        mSpeechSynthesizer.setParam(SpeechSynthesizer.PARAM_SPEAKER, "0"); // 设置在线发声音人： 0 普通女声（默认） 1 普通男声 2 特别男声 3 情感男声<度逍遥> 4 情感儿童声<度丫丫>
        mSpeechSynthesizer.setParam(SpeechSynthesizer.PARAM_VOLUME, "9"); // 设置合成的音量，0-9 ，默认 5
        mSpeechSynthesizer.setParam(SpeechSynthesizer.PARAM_SPEED, "5");// 设置合成的语速，0-9 ，默认 5
        mSpeechSynthesizer.setParam(SpeechSynthesizer.PARAM_PITCH, "5");// 设置合成的语调，0-9 ，默认 5
        mSpeechSynthesizer.setParam(SpeechSynthesizer.PARAM_MIX_MODE, SpeechSynthesizer.MIX_MODE_HIGH_SPEED_SYNTHESIZE);
        mSpeechSynthesizer.setAudioStreamType(AudioManager.STREAM_MUSIC);
        mSpeechSynthesizer.initTts(TtsMode.MIX);
    }

    private void initOfflineResource() {
        try {
            String ettsText = copyAssetsFile("bd_etts_text.dat");
            String ettsSpeech = copyAssetsFile("bd_etts_speech_female.dat");
            mSpeechSynthesizer.setParam(SpeechSynthesizer.PARAM_TTS_TEXT_MODEL_FILE, ettsText); // 文本模型文件路径 (离线引擎使用)
            mSpeechSynthesizer.setParam(SpeechSynthesizer.PARAM_TTS_SPEECH_MODEL_FILE, ettsSpeech);  // 声学模型文件路径 (离线引擎使用)
            if (!audioCacheSp.getBoolean("AuthInfoMixSuccess", false)) {
                AuthInfo authInfo = mSpeechSynthesizer.auth(TtsMode.MIX);
                if (authInfo.isSuccess()) {
                    audioCacheSp.edit().putBoolean("AuthInfoMixSuccess", true).commit();
                }
                Log.v(TAG, "initOfflineResource() authInfo isSuccess = " + authInfo.isSuccess());
            }
        } catch (IOException e) {
            Log.w(TAG, "initOfflineResource() ", e);
        }
    }

    private String copyAssetsFile(String sourceFilename) throws IOException {
        String destFilename = FileUtil.createTmpDir(context) + "/" + sourceFilename;
        FileUtil.copyFromAssets(context.getApplicationContext().getAssets(), sourceFilename, destFilename, false);
        Log.i(TAG, "copyAssetsFile() 文件复制成功：" + destFilename);
        return destFilename;
    }
	```
	
> 注：初始化操作应该放在子线程处理！！！


#### 4. UNIT 理解与交互

文档地址： <http://ai.baidu.com/docs#/UNIT-guide/top>

课程地址： <https://ai.baidu.com/support/video#video-category-unit>

训练中心：<https://ai.baidu.com/unit>

- 对话单元
	- 以词槽为中心，附带澄清话术
	- 当词槽匹配规则时触发答复，答复可以是文本和执行函数
	- 需要大量的数据样本进行训练

- 问答单元
	- 问答多对多模式（多种问法和回答方式）
	- 方便用来写一个闲聊的机器人



