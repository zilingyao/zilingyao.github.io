---
title: 初试MVP
date: 2016/10/4 16:36:52 
categories: Android
tags: 
    - Android
    - 项目
---
## MVP示例
mvp是什么就不多说了，网上详细的介绍很多，直接放效果图来撸代码吧
![enter description here][1]
<!-- more -->

## 准备
### 项目结构
![enter description here][2]


  [1]: http://oegt3xt9o.bkt.clouddn.com/MVP.gif
  [2]: http://oegt3xt9o.bkt.clouddn.com/mvp%E7%BB%93%E6%9E%84%E5%9B%BE.png
### Gradle配置
```java
    compile 'com.zhy:okhttputils:2.6.2'
    compile 'com.google.code.gson:gson:2.4'
```
## MVP
### MVP之V
baseview.java
```java
/**
 * Created by Administrator on 2016/10/10.
 * 业务处理需要的方法  一般就是显示加载框 加载数据 成功失败后隐藏加载框
 */

public interface BaseView {
    void showData(WeatherModelBean mainModelBean);
    void showProgress();
    void hideProgress();
}
```
MainActivity.java
```java
public class MainActivity extends AppCompatActivity implements BaseView {

    private ProgressBar mProgressBar;
    private TextView text;
    private WeatherPresenter mMainPresenter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initView();
    }

    private void initView() {
        text = (TextView) findViewById(R.id.tv_text);
        mProgressBar = (ProgressBar) findViewById(R.id.mProgressBar);
        mMainPresenter = new WeatherPresenter(this);
        //制造延迟效果
        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                mMainPresenter.loadData();
            }
        }, 2000);
    }

    @Override
    public void showData(WeatherModelBean mainModelBean) {
        String data = mainModelBean.getWeatherinfo().getCity()+"\n"
                +mainModelBean.getWeatherinfo().getTime()+"\n"
                +mainModelBean.getWeatherinfo().getCityid()+"\n"
                +mainModelBean.getWeatherinfo().getNjd()+"\n"
                +mainModelBean.getWeatherinfo().getQy()+"\n"
                +mainModelBean.getWeatherinfo().getTemp();
        text.setText(data);
    }

    @Override
    public void showProgress() {
        mProgressBar.setVisibility(View.VISIBLE);
    }

    @Override
    public void hideProgress() {
        mProgressBar.setVisibility(View.GONE);
    }
}
```
### MVP之M
WeatherModelBean.java
```java
public class WeatherModelBean {
    /**
     * 实体类
     */
    private WeatherinfoBean weatherinfo;

    public WeatherinfoBean getWeatherinfo() {
        return weatherinfo;
    }

    public void setWeatherinfo(WeatherinfoBean weatherinfo) {
        this.weatherinfo = weatherinfo;
    }

    public static class WeatherinfoBean {
        private String city;
        private String cityid;
        private String temp;
        private String WD;
        private String WS;
        private String SD;
        private String WSE;
        private String time;
        private String isRadar;
        private String Radar;
        private String njd;
        private String qy;

      //此处省略get set方法
    }
}
```
WeatherModel.java
```java
/**
 * Created by Administrator on 2016/10/10.
 *  业务具体处理，获取数据 操纵数据传给javabean等
 */
public class WeatherModel {
    PMPresenter mPMPresenter;
    public WeatherModel(PMPresenter iMainPresenter) {
        this.mPMPresenter = iMainPresenter;
    }

    public void loadData() {
        String url = "http://www.weather.com.cn/adat/sk/101010100.html";
        OkHttpUtils
                .get()
                .url(url)
                .build()
                .execute(new StringCallback() {
                    @Override
                    public void onError(Call call, Exception e, int id) {
                        mPMPresenter.loadDataFailure();
                    }

                    @Override
                    public void onResponse(String response, int id) {
                        Gson gson = new Gson();
                        WeatherModelBean weatherModelBean = gson.fromJson(response,WeatherModelBean.class);
                        mPMPresenter.loadDataSuccess(weatherModelBean);
                    }
                });
    }
}
```
### MVP之P
basePresenter.java
```java
//此接口是基类presenter  添加view 移除view
public interface BasePresenter<V> {

    void attachView(V view);

    void detachView();
}
```
pmPresenter.java
```java
/**
 * Created by Administrator on 2016/10/10.
 * 此接口 作用 连接presenter和model
 */

public interface PMPresenter {
    void loadDataSuccess(WeatherModelBean mainModelBean);
    void loadDataFailure();
}
```
WeatherPresenter.java
```java
public class WeatherPresenter implements BasePresenter<BaseView>,PMPresenter{
    private BaseView mMainView;
    private WeatherModel mMainModel;
    public WeatherPresenter(BaseView view) {
        attachView(view);
        mMainModel = new WeatherModel(this);
    }
    @Override
    public void attachView(BaseView view) {
        this.mMainView = view;
    }
    @Override
    public void detachView() {
        this.mMainView = null;
    }
    public void loadData() {
        mMainView.showProgress();
        mMainModel.loadData();
    }
    @Override
    public void loadDataSuccess(WeatherModelBean mainModelBean) {
        mMainView.showData(mainModelBean);
        mMainView.hideProgress();
    }
    @Override
    public void loadDataFailure() {
        mMainView.hideProgress();
    }

}
```
## 总结
### MVC模式

视图（View）：用户界面。
控制器（Controller）：业务逻辑
模型（Model）：数据保存
View 传送指令到 Controller
Controller 完成业务逻辑后，要求 Model 改变状态
Model 将新的数据发送到 View，用户得到反馈

### MVP模式
使用MVP时，Activity和Fragment变成了MVC模式中View层，Presenter相当于MVC模式中Controller层，处理业务逻辑。每一个Activity都有一个相应的presenter来处理数据进而获取model。

### MVVM模式
将 Presenter 改名为 ViewModel，基本上与 MVP 模式完全一致。唯一的区别是，它采用双向绑定（data-binding）：View的变动，自动反映在 ViewModel，反之亦然。

   	
