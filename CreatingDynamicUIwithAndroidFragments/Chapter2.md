# 灵活的碎片和UI

> - 主题
>
>   > - 精简设备差异化的问题
>   > - 动态资源选择
>   > - 协调碎片内容
>   > - FragmentManager的作用
>   > - 跨Activities支持碎片

## 2.1创建灵活的UI

> - 可以根据运行的具体设备灵活的重排碎片。

### 2.1.1动态的碎片布局选择

> - 尽可能降低用户界面和应用程序代码之间的依赖程度；尽可能在布局资源中做更多与用户界面相关的工作。
>
> - 碎片化使得修改布局变得简单，Android系统会根据设备差异性选择合适的碎片布局，减少重复的代码，使得应用程序的维护变得简单。
>
> - 可以根据屏幕朝向，屏幕大小动态布局碎片。
>
> - 使用资源**屏幕大小限定符**将资源与特定的屏幕大小特征关联。
>
> - 管理屏幕大小的时候，Android使用规范的单位--------**dp**(density-independent pixel，密度无关像素)。
>
> - 因为dp单元始终对用160dpi设备上物理像素的大小，所以它可以提供与设备物理像素大小无关的一致测量单元。
>
> - Android平台会维护在dp与物理像素之间的映射。
>
> - Android提供三种屏幕大小**限定符**：
>
>   > - 最小屏幕宽度大小限定符：对应屏幕最窄地方的dp，与设备朝向无关。**改变设备方向并不会改变设备最小宽度**。选项名称：**Smallest screen width**。
>   > - 可用屏幕宽度大小限定符：对应当前屏幕朝向从左到右的dp。**改变设备方向会改变可用屏幕宽度**。选项名称：**Screen width**。
>   > - 可用屏幕高度大小限定符：与可用屏幕宽度大小限定符相识。选项名称：**Screen height**。
>
> - **消除冗余布局的描述**：
>
>   > - 使用布局别名(layout aliasing)，来减少布局资源的重复。
>
> - **布局别名**：
>
>   > - 创建一个不加限定符的布局文件，比如activity_main_wide.xml
>   >
>   > - 将带限定符的布局内容｛比如：activity_main.xml (land)｝转移至该文件。
>   >
>   > - 修改activity_main.xml (land)
>   >
>   >   ```xml
>   >   <merge>
>   >   	<include layout="@layout/activity_main_wide"/>
>   >   </merge>
>   >   ```
>
> - **设计灵活的碎片**：
>
>   > - 因为碎片间相关性大，很少有碎片与其他碎片完全对立。
>   > - 在相应的碎片的类中处理相应的表现。
>
> - **拒绝高耦合**：
>
>   > - 比如“选择按钮碎片”中的点击事件会改变“内容显示碎片”中的显示内容。在界面只有“选择按钮碎片”情况下，也会对“内容显示碎片”操作；改动“内容显示碎片”也可能会影响“选择按钮碎片”，过高的耦合会给程序带来诸多问题。
>
> - **抽象碎片间的联系**
>
>   > - 避免创建碎片间的直接联系，我们可以通过接口实现抽象。
>   >
>   > - **回调接口定义**：
>   >
>   >   > - 接口应该是面向应用级（比如：选择一本书）而非实现级别的（比如：点击选项按钮）。实现级别的操作应该隔离在碎片中。
>   >   >
>   >   > - 参数的定义不要过于局限。比如：定义为书名就很局限，定义为index就很灵活。
>   >   >
>   >   >   ```java
>   >   >   public interface OnSelectedBookChangeListener{
>   >   >   	void onSelectedBookChanged(int bookIndex);
>   >   >   }
>   >   >   ```
>   >
>   > - **使碎片自包含**：
>   >
>   >   > - Fragment类应该隐藏用户选择的细节，并将每个选择转换为图书标识符。
>   >
>   > - **碎片通知**:
>   >
>   >   > - 碎片可以通过getActivity方法找到放置该碎片的活动。
>   >   > - 获取到Activity后映射到抽象的接口
>   >
>   > - **封装碎片操作**
>   >
>   > - **创建碎片间松散的连接关系**
>   >
>   >   > - 书中代码异常修改
>   >   >
>   >   >   ```java
>   >   >   //书中代码
>   >   >   FragmentManager fragmentManager =getFragmentManager();
>   >   >   
>   >   >   //更正为
>   >   >   FragmentManager fragmentManager=getSupportFragmentManager();
>   >   >   
>   >   >   //来源
>   >   >   import androidx.fragment.app.FragmentManager;
>   >   >   ```
>   
> - **碎片可以防止意外**
>
>   > - 如果我们需要适配不同设备，比如平板和手机。平板中可能是两个碎片显示在一个界面，而手机是将两个碎片分布在两个Activity中。平板中两个碎片的的交互可以通过抽象接口实现，手机中可以通过Intent方法intent.putExtra()携带信息。
>   >
>   >   ```java
>   >   if(bookDescFragment == null || !bookDescFragment.isVisible()){
>   >   	// Use activity to display description
>   >   	if(!mCreating) {
>   >   		Intent intent = new Intent(this, BookDescActivity.class);
>   >   		intent.putExtra("bookIndex", bookIndex);
>   >   		startActivity(intent);
>   >   	}
>   >   }else {
>   >   	// Use contained fragment to display description
>   >   	bookDescFragment.setBook(bookIndex);
>   >   }
>   >   ```
>   >
>   > - **上述代码重点**：我们判断的条件是两个，碎片的引用是不是null和碎片是否可见，这是因为在某种情况下，我们将设备由横向切换至竖向时，FragmentManager仍然可以引用到在横向时候的那个碎片(即使现在它并不可见)。
>   >
>   >   ```java
>   >   public class MainActivity extends Activityimplements OnSelectedBookChangeListener{
>   >   	boolean mCreating = true;
>   >   	@Override
>   >   	protected void onResume() {
>   >   		super.onResume();
>   >   		mCreating = false;
>   >   	}
>   >   	// other members elided for clarity
>   >   }
>   >   ```
>   >
>   > - **上述代码重点**：Android在手机横向纵向切换的时候会重新创建Activity实例，然后Android系统去恢复用户之前的选择时，会触发onSelectedBookChanged方法。因此我们要避免在创建阶段调用。

