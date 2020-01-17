# Android四大组件必知必会

## 一、Activity

#### # Activity的生命周期？

正常情况下，Activity的生命周期是：

`onCreate` - `onStart` - `onResume` - `onPause` - `onStop` - `onDestroy`

Activity A跳转到Activity B：

`A-onCreate` - `A-onStart` - `A-onResume` - `A-onPause` - `B-onCreate` - `B-onStart` - `B-onResume` - `A-onStop` 

接着finish掉Activity B，这个时候的生命周期为：

`B-onPause` - `A-onRestart` - `A-onStart` - `A-onResume` - `B-onStop` - `onDestroy`

#### # 一些特殊情况下的Activity的生命周期？横竖屏切换？

#### # Fragment的生命周期？

#### # Activity的四大启动模式？

## 2. Service

#### # 两种Service的使用？以及生命周期

#### # IntentService

### 3. 广播

#### # 有序广播和无序广播

#### # 动态广播和静态广播

### 4. 内容提供者

