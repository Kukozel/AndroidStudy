# Fragment的生命周期和定制化

> - 碎片设置/显示事件的序列
> - 碎片销毁/隐藏事件的序列
> - **DialogFragment**的利用

## 3.1理解碎片的声明周期

> - 碎片要与Activity的生命周期配合，他们密切关联。
> - 碎片在设置/显示/销毁/隐藏时段提供许多与Activity生命周期相关的回调方法。
> - 碎片也提供与碎片中包含的Activity生命周期相关的回调方法。

### 3.1.1碎片的setup和display

> - **理解碎片的设置和现实**
>
>   > - 碎片的设置和展示是一个多阶段的过程，包含：
>   >
>   >   > - 碎片与Activity的关联。
>   >   > - 碎片创建
>   >   > - 推动Activity进入活动状态的标准生命周期事件。
>   >
>   > - ![1565243196229](C:\Users\Abfahrt\Documents\MdNotes\images\碎片与活动设置展示阶段生命周期.png)
>   >
>   >   > - 1.Activity的**onCreate**回调方法中的**setContentView**方法会载入布局资源，并触发Activity与fragments的关联。
>   >   > - 2.**注意**：在fragment创建前，便将fragment附属于Activity了。碎片首先收到一个附属通知，然后在**onAttach**方法中收到一个Activity的引用。
>   >   > - 3.之后，Activity也会收到通知，然后在**onAttachFragment**方法中收到一个fragment的引用。
>   >   > - 4.在**碎片创建前便与Activity创建了附属关系**是有意义的。在许多情况下，因为Activity通常包含碎片将显示的信息或碎片创建过程中所需重要的信息，所以碎片需要在创建过程中访问活动。
>   >   > - 5.碎片附属于Activity后，碎片在**onCreate**方法中执行一般的创建工作，然后在**onCreateView**方法中构造其所包含的视图层次结构。
>   >   > - 6.当一个活动包含多个碎片时，Android依次调用一个碎片的**Fragment.onAttach**,**Activity.onAttachFragment**,**Fragment.onCreate**, **Fragment.onCreateView**方法后再去为下一个碎片调用这些方法。一个一个按顺序。
>   >   > - 7.完成所有碎片调用上述四个方法后，再依次为每个片段分别调用设置和显示回调方法中的其余部分。
>   >   > - 8.Activity完成onCreate方法的执行后，Android调用每个碎片的**onActivityCreated**方法。onActivityCreated方法指出，由Activity的布局资源创建的所有视图和碎片都已完全构造好，并且可以安全地访问。
>   >   > - 9.之后，碎片在Activity的**onStart**，**onResume**方法后执行标准的生命周期回调**onStart**，**onResume**。看起来就像是在Activity的**onStart**里执行碎片的**onStart**，**onResume**同理。
>   >   > - 10.对于大多数的碎片而言，我们只是重写**onCreate**和 **onCreateView** 方法。

### 3.1.2避免方法名称的混淆

> - Android调用fragment类的**onCreateView**方法是用来给fragment一个时机，用来创建和返回一个fragment所包含的视图层次结构。

### 3.1.3碎片的hide和teardown

> - 理解碎片的
>
>   > ![1565314416001](C:\Users\Abfahrt\Documents\MdNotes\images\碎片与活动隐藏与销毁阶段生命周期.png)
>   >
>   > - **onDestroyView**方法销毁**onCreateView**返回的视图层级后，碎片的**onDestroy**再被执行，最后**onDetach**，之后碎片与**Activity**不再关联，**getActivity**方法返回**null**。
>   > - 多个碎片销毁时依次执行7->8->9，这之后再调用**onDestroy**方法。

### 3.1.4最大化最大化可用资源

> - **碎片**将它的创建与销毁与其包含的视图层次结构分开。这是因为**碎片**能够在没有视图层次结构的情况下存在并与**Activity**关联。
> - 一个Activity可能包含多个碎片，特定时间内只显示几个碎片，所有在显示时候调用**onCreateView**，在不显示时会调用**onDestroyView**。

### 3.1.5管理碎片状态

> - 可以在**onSaveInstanceState**方法中持久保存碎片的状态信息，之后在**onCreate**和**onCreateView**方法中将该状态传递回碎片。
> - 代价高的资源初始化(例如：连接数据源、复杂计算或资源分配)应该在**onCreate**中，而不是**onCreateView**中，这样可以避免重复载入。

## 3.2 特殊目的的碎片类

> - 许多特殊类会影响在生命周期的各个点上执行哪些操作是否安全，甚至其中一些类会添加了它们自己的生命周期的方法。

### 3.2.1 ListFragment

> - **ListFragment**类是使用最简单的，帮助效果好的碎片派生类之一。
>
> - **ListFragment**类提供了一个封装ListView的fragment，对于显示数据列表非常有用。
>
> - **将数据与列表关联**
>
>   > - **ListFragment**不需重写**onCreateView**，只需关联数据。**ListFragment**类自己完成了创建视图层次结构和显示该数据的所有工作。
>   > - 关联数据只需调用**ListFragment**类的**setListAdapter**方法，并将引用传递给实现**ListAdapter**接口的对象。Android提供了许多实现这个接口的类，比如ArrayAdapter、SimpleAdapter、SimpleCursorAdapter。
>   > - **ListFragment**类封装了**ListView**类的一个实例，该实例可以通过**getListView**方法访问。
>   > - **注意**：一个例外是：创建ListAdapter实例时。ListFragment和ListView类都公开一个setListAdapter方法，但必须确保使用**ListFragment**版本的方法。
>
> - **从显示中分离数据**
>
> - **处理ListFragment选择项**
>
>   > - 可以使用接口来送碎片与Activity之间的耦合。



### 3.2.2 DialogFragment

> - **DialogFragment**类里面有与对话框相关的许多特殊处理。