# Chapter1:碎片和UI模块化

> - 主题：
>
>   > - 一种新的UI方式
>   > - 碎片化

## 一种新的UI方式

> - 以前Android设备差异化小，直接在Activity中绘制界面就可以。
> - 现在设备差异化大，通过创建单一的Activity，实现在各种差异化设备上统一管理用户界面，是非常难的。
> - 对不同的种类设备分类然后创建各自的Activities并不是一个好的方法，会增加大量的冗余代码，也会使得代码逻辑更为复杂。Google's material design标准会使得每个Activity中的代码更为复杂。
> - **碎片化**是一种更好的解决方式，可以将我们的用户交互界面模块化成小部分，在需要的时候进行组合。
> - **碎片化**可以将用户界面按照组成和逻辑分组，Activity可以根据需要为具体的设备安排碎片。
> - 碎片关注设备细节，Activity关注交互界面总体。

### 1.1平台架构对碎片的广泛支持

> - Android平板的出现使得UI设计变得更难。
>
> - 碎片化的方式不失为一种良好解决的方式，根据用户交互界面的组成和逻辑进行高效的分组。
>
> - 95%的设备支持碎片化，另外低版本的5%也可以通过 v4 的库来支持碎片化。
>
>   > - 参考地址：http://developer.android.com/tools/support-library/index.html

### 1.2碎片化如何简化Android任务

> - 碎片化不光简化我们创造用户交互界面，而且会简化Android内置的用户交互任务。
> - **规整化碎片开发**使得我们可以使用统一方法开发解决问题。

### 1.3Fragments和Activities之间的关系

> - fragments不是取缔Activities的，而是对Activities的增补；fragment存在于Activity之中。
> - 一个Activity实例可以包含多个fragment，但是一个fragment只能存在于一个activity。
> - fragment与包含它的activity密切相关，生命周期耦合度也极高。
> - 不要过度应用碎片，并非每个Activity都要包含碎片。
> - 使用碎片化会使程序变得复杂，没价值的情况下不要使用。

### 1.4向碎片化转移

> - 大部分情况下一个碎片是从**布局资源**中创建，但有时也可以和Activitis一样，通过**编程方式**创建。
> - 就如同创建Activity一样，从布局文件创建碎片要遵循相同的规则和技术。
> - 使用碎片与以往最大的不同就是：找到一个能将用户交互界面的布局划分成便于管理的小部分。
> - 将传统的以Activity为导向的用户界面转化成碎片化是学习碎片化的最简单的方式。

### 1.5关Activity为导向的旧式开发的思考

> - 很常规，略。

### 1.6关于碎片化为导向的新式开发的思考

> - 要合理的将用户交互界面分片。
>
> - 最小化假设：
>
>   > - 我们为碎片设置越少的设定，碎片的可复用性越高。
>   > - 比如我们将一个包含**ScrollView**的碎片的layout_height属性值设为 0，在指定碎片明确大小的场景下就不能正常显示。
>   > - 最好的方式是让碎片完全占据分配给他的位置，这样碎片对自身的位置和大小具有最佳的控制权。
>   > - 将layout_height 属性值设为match_paren，并指定namespace：xmlns:android="http://schemas.android.com/apk/res/android"

### 1.7创建Fragment类

> - 所有的fragment类都要继承android.app.Fragment类。
>
> - 重载方法中最重要的是onCreateView方法。该方法需要三个参数，我们只需要注意前两个参数：
>
>   > - **inflater**：这是一个**布局填充器**（LayoutInflater）实例的引用，可以在包含Activity的上下文中读取和扩展布局资源。
>   > - **container**：这是一个活动部局中的**视图组**（ViewGroup）实例的引用，也是这个碎片的视图层次要被添加的地方。
>
> - LayoutInflater提供inflater方法，该方法处理将布局资源转换为相应视图层次结构的细节，并返回对该层次结构根视图的引用。
>
> - 使用LayoutInflater.inflater方法，可以将MyFragment类的onCreateView方法构造，并返回与R.layout.fragment_my_example布局资源相对应的视图层次结构。
>
> - LayoutInflater.inflater方法的第三个参数false，指明**container**（容器）仅被用于layout的参数；如果设置为true，则会在onCreateView方法中将新的视图层级添加到视图组中。我们不需要这样做，因为要在Activity中处理它。

### 1.8其他

> - **tools:layout**：在预览设计器中的布局，run之后无用。

































