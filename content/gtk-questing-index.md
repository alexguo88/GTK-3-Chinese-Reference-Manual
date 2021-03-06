## 常见问题
常见问题——寻找GTK+手册中常见问题的答案

## 问题和答案
这是一个用常见“How do I...“组织的参考手册”索引“。如果你不确定去读哪个手册来解决你的问题，这个列表是个开始的好地方。

##### 常规
###1.1
**我如何在GTK+上起步？**

GTK+[网站](www.gtk.org)提供了一些[教程](www.gtk.org/documentation.php)和其他文档（大部分是关于GTK+ 2.x的，但大部分依然可用）。更多的从文本资源到在线书籍可以在[GNOME developer's site](https:/developer.gnome.org)中找到。在学习这些材料后，你应该准备回到这个索引来了解细节。
###1.2
**我可以在哪里得到GTK+的帮助，提交bug报告或者制作一个特性请求？**

参阅[此文档顶部](gtk-resources.md)
###1.3
**我如何从一个GTK+版本迁移到另一个？**

参阅[Migrating from GTK+2.x to GTK+ 3](https://developer.gnome.org/gtk3/3.14/gtk-migrating-2-to-3.html)。你也许能在这个文档里得到关于某些部件和函数的有用信息。
如果你有一个手册里所没涵盖的问题，请在邮件列表上提问然后对于文档在[提交一个bug报告](https://bugzilla.gnome.org)。
###1.4
**GTK+中的内存管理如何工作？程序返回时我是否要释放数据？**

查看[GObject](https://developer.gnome.org/gobject/unstable/gobject-The-Base-Object-Type.html#GObject)和[GInitiallyUnowned](https://developer.gnome.org/gobject/unstable/gobject-The-Base-Object-Type.html#GInitiallyUnowned)文档。对于GObject特别主义g_object_ref()和g_object_unref()。GInitiallyUnowned是GObject的子类，所以同样注意，特别对于有"floating“状态的（相应文档中有解释）。
对于函数返回的strings，如果不应该被释放它们要被声明为"const"。非"const"的strings应该被g_free()释放。Arrays遵从同样的规则。如果你发现一个没被记录的特别是规则，请向[http://bugzilla.gnome.org](http://bugzilla.gnome.org)提交bug。
###1.5
**当我在创建部件后立即销毁了它，为什么我的程序还会泄露内存？**

如果GtkFoo 不是个顶层窗体，然后
```c
foo = gtk_foo_new();
gtk_widget_destroy(foo);
```
是一个内存泄露，因为没有东西假定首次浮动引用，如果你正在使用一个部件而且你没有立即将它填充进一个容器，然后你想要正常的引用计数，而不是浮动引用计数。

为了做到这些，你必须获得一个部件的参考并且在创建它之后丢弃引用参考（"ref and sink"in GTK+ parlance）。
```c
foo = gtk_foo_new ();
g_object_ref_sink (foo);
```
当你想要丢弃部件，你必须在丢弃引用前调用`gtk_widget_destroy()`来破坏部件的任何外部连接
```c
gtk_widget_destroy (foo);
g_object_unref (foo);
```
当你立即向容器添加一个部件，它安排假设一个首次浮动引用所以你不必担心引用计数……只要调用`gtk_widget_destroy()`丢弃该部件即可。
###1.6
**我该如何在GTK+中使用线程？**

这在GDK线程文档中有说明。也可参看[GThread](https://developer.gnome.org/glib/unstable/glib-Threads.html)文档的便携线程原型。
###1.7
**我该如何国际化一个GTK+程序？**

大多数人使用[GNU gettext](www.gnu.org/software/gettext/)，它在安装GLib时被要求。在一个已安装gettext的UNIX或者Linux系统，键入info gettext阅读文档。

使用gettext的简短清单：调用bindtextdomain()所以gettext能找到包括译文的文件，调用textdomain()来设定默认翻译区域，调用bind_textdomain_codeset()来要求所有已翻译的字符串以UTF-8返回，然后调用gettext()来查找每个字符并翻译为默认区域文字。

gi18n.h提供了以下简略的宏。按照惯例，人们定义了以下的宏。
```c
#define _(x)		gettext (x)
#define N_(x)		x
#define C_(ctx,x)	pgettext(ctx, x)
```
用`N_()`（没有操作数）在译文中标记一个字符串，字符串所在的位置不允许调用gettext()，比如在一个数组初始化程序中。你最终必须在字符串上调用gettext()来获取译文。`_()`标记了译文的字符串并且实际上翻译了它。`C_()`宏向标记的译文增加一个额外的上下文(context)，这可以消除了短字符串在程序的不同部分需要不同翻译而产生的歧义。

使用完这些宏的代码如下：
```c
#include <gi18n.h>
static const char *global_variable = N_("Translate this string");

static void
make_widgets (void)
{
	GtkWidget *label1;
	GtkWidget *label2;

	label1 = gtk_label_new (_("Another string to translate"));
	label2 = gtk_label_new (_(global_variable));
...
```
库应该使用dgettext()而不是gettext()，gettext()允许每次翻译时指定翻译区域。库也应该避免调用textdomain()，因为它会指定区域而不是使用默认的区域。

按照宏GETTEXT_PACKAGE被定义去控制你的库的翻译区域的惯例，gi18n-lib.h可以被包括来提供以下便利：
```c
#define _(x) dgettext (GETTEXT_PACKAGE, x)
```
###1.8
**我该如何在GTK+程序中使用noo_ASCII字符？**

GTK+对文本使用Unicode（更确切的说是UTF-8），UTF-8把每个Unicode值译成1到6个字节，而且有一些能使在C程序中使用Unicode文本是个不错选择的良好特性：
+ ASCII 字符被译为相近的ASCII码值
+ ASCII 字符不会作为其他字符的一部分出现
+ 零字节不会作为一个字符的一部分出现，所以UTF-8字符串可以被常见的c库函数用来控制零结尾字符串

更多关于Unicode和UTF-8的信息可以在[UTF-8 and Unicode FAQ for Unix/Linux](www.cl.cam.ac.uk/~mgk25/unicode.html)，Glib提供UTF-8和其他编码间字符串转换的函数，参看[g_locale_to_utf8](https://developer.gnome.org/glib/unstale/glib-Character-Set-Conversion.html#g-locale-to-utf8)和[g_convert()](https://developer.gnome.org/glib/unstable/glib-Charater-Set-Conversion.html#g_convert)。

外部来源的文本（比如文件或者用户输入)，在被传递给GTK+前必须被转换为UTF-8。下面的例子将一个ISO-8859-1编码的文本内容写出到标准输出：
```c
gchar *text, *utf8_text;
gsize length;
GError *error = NULL;

if (g_file_get_contents (filename, &text, &length, NULL))
  {
     utf8_text = g_convert (text, length, "UTF-8", "ISO-8859-1",
                            NULL, NULL, &error);
     if (error != NULL)
       {
         fprintf ("Couldn't convert file %s to UTF-8\n", filename);
         g_error_free (error);
       }
     else
       g_print (utf8_text);
  }
else
  fprintf (stderr, "Unable to read file %s\n", filename);
```
对于源码中的字符串，这里有几个对待non-ASCII内容的方案：
+ 直接用UTF-8
+ 逃离UTF-8
+ 运行时转换

这里有一个范例显示了三种使用版权标志&copy 的方法，该标志有Unicode和ISO-8859-1码值169和在UTF-8中被替代为194,169两个字节，或者作为字符串的"\302\251"。
```c
g_print ("direct UTF-8: ©");
g_print ("escaped UTF-8: \302\251");
text = g_convert ("runtime conversion: ©", -1, "ISO-8859-1", "UTF-8", NULL, NULL, NULL);
g_print(text);
g_free (text);
```

如果你使用gettext()本地化你的应用程序，你必须调用bind_textdomain_codeset()来确定被翻译字符串以UTF-8返回。
###1.9
**我该如何在GTK+中使用C++？**

有两个方法来实现它。GTK+头文件使用了C的子集它也兼容C++，所以你可以在C++程序中使用正常的GTK+ API。另外，你可以使用一个"C++版本"比如提供原生C++ API的[gtkmm](gtkmm.sourceforge.net)。

当直接使用GTK+时，注意只有函数可以和信号连接，不是方法。所以你必须使用全局函数或者"静态"类函数来连接信号。

另外一个直接使用GTK+常见的问题是C++不会隐含的把一个整型转换为枚举。当使用位域(bitfields)时会发生以上情况；在C中你可以写以下代码：
```c
gdk_window_set_events (gdk_window,
						GDK_BUTTON_PRESS_MASK | GDK_BUTTON_RELEASE_MASK);
```
在C++中必须写：
```
gdk_window_set_events (gdk_window,
						(GdkEventMask) GDK_BUTTON_PRESS_MASK | GDK_BUTTON_RELEASE_MASK);
```
然而，很少有函数需要这个。
###1.10
**我该如何在GTK+中使用其他non_C languages？**

参看[list of language bindings](www.gtk.org/language-bindings.php)和[http://www.gtk.org](http://www.gtk.org)。
###1.11
**我该如何从从文件中加载一副图片或者动画？**

用gtk_image_new_from_file()来将一个图片文件直接载入显示部件。用gdk_pixbuf_new_from_file()载入另外目的的图片。用gdk_pixbuf_animation_new_from_file()载入动画。gdk_pixbuf_animation_new_from_file也可以载入非动画图片，将它与gdk_pixbuf_animation_is_static_image()结合使用可以载如未知类型文件。

用GdkPixbufLoader异步载入一个图片或者动画文件（非堵塞）。
###1.12
**我该如何绘制文字？**

绘制文字，你可以使用Pango layout和pango_cairo_show_layout()。
```c
layout = gtk_widget_create_pango_layout (widget, text);
fontdesc = pango_font_description_from_string ("Luxi Mono 12");
pango_layout_set_font_description (layout, fontdesc);
pango_cairo_show_layout (cr, layout);
pango_font_description_free (fontdesc);
g_object_unref (layout);
```
###1.13
**我该如何测量一段文字的大小？**

为了掌握一段文字的大小，你可以使用Pango layout和pango_layout_get_pixel_size()，向下面这样使用代码：
```c
layout = gtk_widget_create_pango_layout (widget, text);
fontdesc = pango_font_description_from_string ("Luxi Mono 12");
pango_layout_set_font_description (layout, fontdesc);
pango_layout_get_pixel_size (layout, &width, &height);
pango_font_description_free (fontdesc);
g_object_unref (layout);
```
###1.14
**为什么当我用GTK_TYPE_BLAH 宏时，类型没注册？**

GTK_TYPE_BLAH 宏被定义就像调用 gtk_lah_get_type()，_get_type()函数被描述为当它的值为被使用时允许编译器优化调用（call away）的G_GUNC_CONST。

这个问题的通常应变方法是在一个易变变量里储存结果，它使编译器优化调用。
```c
volatile GType dummy = GTK_TYPE_BLAH;
```

###1.15
**我该如何创建一个透明的顶层窗口？**

使一个窗体透明需要用一个支持透明的视觉资料。它通过使用gdk_screen_get_rgba_visaul()得到屏幕的RGBA视频资料并且将它设置在窗体上。**注意**如果屏幕不支持透明窗口gdk_screen_get_rgba_visaul()会返回NULL，你应该退回到gdk_screen_get_rgba_visaul()。另外，**注意**从一个屏幕移到另一个屏幕，它会改变，所以不管什么时候窗体被移动到另一个窗体它需要被重复。
```c
GdkVisual *visual;

visual = gdk_screen_get_rgba_visual (screen);
if (visual == NULL)
	visual = gdk_screen_get_system_visual (screen);
gtk_widget_set_visual (GTK_WIDGET (window), visual);
```
简单的使用cairos RGBA绘制属性来填充窗体的alpha通道。

**注意**一个RGBA视觉资料的出现并不保证窗体会透明出现在屏幕上。在X11上，它需要一个正在运行的复合管理(compositing manager)。参见gtk_widget_is_composited()寻找alpha通道是否被支持。 
##### 我该用哪个部件……
###2.1
**对于列表和树？**

参看[tree widget overview](https://developer.gnome.org/gtk3/3.14/TreeWidget.html)——你应该使用[GtkTreeView](https://developer.gnome.org/gtk3/3.14/GtkTreeView.html)。（列表是个没有分支的树，所以树部件对于列表同样有用）。
###2.2
**对于多行文本显示和编辑？**

参看[text widget overview](https://developer.gnome.org/gtk3/3.14/TextWidget.html)，你应该使用[GtkTextView](https://developer.gnome.org/gtk3/3.14/GtkTextView.html)。

如果你只有少量的文本，`GtkLabel`当然更合适。可以用`gtk_label_set_selectable()`来使它可选。对于单行文本输入，见[GtkEntry](https://developer.gnome.org/gtk3/3.14/GtkEntry)。
###2.3
**为了显示一张图像或者动画？**

`GtkImage`仅能显示GTK+能理解的格式的图像。如果你需要做更复杂的事，比如在图像上绘制文本或者图形，可以用`GtkDrawingArea`。
###2.4
**对于呈现一个互斥选项的集合，在窗口哪里用`combo box`？**

对于这种情况，建议用`GtkComboBox`部件。这个部件看起来像combo box或current option menu，取决于当前主题。如果你需要一个可编辑文本，用`has-entry`属性。
##### GtkWidget
###3.1
**我如何改变部件的颜色？**

参看`gtk_widget_override_color()`和`gtk_widget_override_background_color()`。你可以通过安装一个自定义style provider来改变部件外观。参看`gtk_style_context_and_provider()。`

改变部件的背景颜色比如没有背景的`GtkLabel`，可以把他们放在`GtkEventBox`中然后设置event box的背景。
###3.2
**我该如何改变部件的字体?**

这有几个可行的答案，它们取决于你具体想实现的。一个选择是`gtk_widget_override_font()`。

```c
PangofontDesc *font_desc = pango_font_description_new ();
pango_font_description_set_size (font_desc, 40);
gtk_widget_override_font (widget, font);
pango_font_description_free (font_desc);
```

如果你想使文本更大，可以使用`gtk_label_set_markup()`：

```c
gtk_label_set_markup (label, "<big>big text</big>");
```

它更适合许多应用程序，因为它对于用户字体尺寸选择是个相对的尺寸，参看`g_markup_escape_text()`，如果你想灵活的构建。
你也可以用以下方法来改变部件字体
```
.my-widget-class {
	font: Sans 30;
}
```
在CSS文件中，通过`gtk_css_provider_load_from_file()`加载它，用`gtk_style_context_add_provider_for_screen()`添加provider。用`gtk_style_context_add_class`在`GtkStyleContext`设定style class，可以将style信息和你的控件联系在一起。这个方法的优点是你可以重新覆盖你选择的字体。参见`GtkStyleContext`了解更多。
###3.3
**我该如何禁用一个部件？**

在GTK+中，一个失效部件被称做"insensitive"参看`gtk_widget_set_sentitive()`。
##### GtkTextView
###4.1
**我该如何得到输入文本的内容并将它作为string？**

参看`gtk_text_buffer_get_bounds()`和`gtk_text_buffer_get_text()`或`gtk_text_iter_get_text()`

```c
GtkTextIter start, end;
GtkTextBuffer *buffer;
char *text;

buffer = gtk_text_view_get_buffer (GTK_TEXT_VIEW (text_view));
gtk_text_buffer_get_bounds (buffer, &start, &end);
text = gtk_text_iter_get_text (&start, &end);
/*use text*/
g_free (text);
```
###4.2
**我该如何使一个文本部件完全用某种字体显示它的内容？**

如果你使用`gtk_text_buffer_insert_with_tags()`来添加选择字体的tags，输入文本会有想要的表现，但是在tag块前后的用户输入文本会以默认style显示。
为保证所有文本都有想要的表现，用`gtk_widget_override_font()`来改变部件的默认字体。
###4.3
**我该如何是text view自动滚动到buffer最后？**

一个使text view自动滚到到尾部的好方法是在buffer最后安排一个mark，并且给它一个right gravity。gravity可以影响在mark上输入的文本，使其保持在尾部。
为了确保buffer尾部可见，在输入新文本后用`gtk_text_view_scroll_to_mark()`滚动到mark。
##### GtkTreeView
###5.1
**我该如何将一些数据和tree的某一行联系起来？**

记住，`GtkTreeModel`的列不一定得显示出来。所以你可以把非用户可见数据像其他数据一样放在model里，通过`gtk_tree_modle_get()`取回它，参看[tree widget overview](https://developer.gnome.org/gtk3/3.14/TreeWidget.html)
###5.2
**我该如何把图片和文本放进同一列？**

你可以用`gtk_tree_view_column_pack_start`或`gtk_tree_view_column_pack_end`将不止一个`GtkCellRenderer`放进单个`GtkTreeViewColumn`。如此便能把一个`GtkCellRendererPixbuf`和一个`GtkCellRendererText`放入一列。
###5.3
**我能用`Gtk_list_store_set()`和`gtk_tree_store_set()`轻松的将数据放在`GtkTreeStore`/`GtkListStore`上，但无法读回？**

`GtkTreeStore`和`GtkListStore`都实现`GtkTreeModel`接口。必然地，你可以使用这个接口实现的任何函数。使用`gtk_tree_model_get()`读回数据是个好办法。
###5.4
**我该如何改变由`GtkTreeView`定义的行列数？**

用`gtk_tree_view_insert_column_with_data_func()`或`gtk_tree_column_set_cell_data_func()`然后自己做从number到string的转换（比如`g_strdup_printf()`）。
下面的例子显示了这个方法：

```c
enum
{
  DOUBLE_COLUMN,
  N_COLUMNS
};

GtkListStore *mycolumns;
GtkTreeView *treeview;

void
my_cell_double_to_text (GtkTreeViewColumn *tree_column,
                    GtkCellRenderer   *cell,
                        GtkTreeModel      *tree_model,
                    GtkTreeIter       *iter,
                        gpointer           data)
{
  GtkCellRendererText *cell_text = (GtkCellRendererText *)cell;
  gdouble d;
  gchar *text;

  /* Get the double value from the model. */
  gtk_tree_model_get (tree_model, iter, (gint)data, &d, -1);
  /* Now we can format the value ourselves. */
  text = g_strdup_printf ("%.2f", d);
  g_object_set (cell, "text", text, NULL);
  g_free (text);
}

void
set_up_new_columns (GtkTreeView *myview)
{
  GtkCellRendererText *renderer;
  GtkTreeViewColumn *column;
  GtkListStore *mycolumns;

  /* Create the data model and associate it with the given TreeView */
  mycolumns = gtk_list_store_new (N_COLUMNS, G_TYPE_DOUBLE);
  gtk_tree_view_set_model (myview, GTK_TREE_MODEL (mycolumns));

  /* Create a GtkCellRendererText */
  renderer = gtk_cell_renderer_text_new ();

  /* Create a new column that has a title ("Example column"),
   * uses the above created renderer that will render the double
   * value into text from the associated model's rows.
   */
  column = gtk_tree_view_column_new ();
  gtk_tree_view_column_set_title  (column, "Example column");
  renderer = gtk_cell_renderer_text_new ();
  gtk_tree_view_column_pack_start (column, renderer, TRUE);

  /* Append the new column after the GtkTreeView's previous columns. */
  gtk_tree_view_append_column (GTK_TREE_VIEW (myview), column);
  /* Since we created the column by hand, we can set it up for our
   * needs, e.g. set its minimum and maximum width, etc.
   */
  /* Set up a custom function that will be called when the column content
   * is rendered. We use the func_data pointer as an index into our
   * model. This is convenient when using multi column lists.
   */
  gtk_tree_view_column_set_cell_data_func (column, renderer,
                                       my_cell_double_to_text,
                                           (gpointer)DOUBLE_COLUMN, NULL);
}
```
###5.5
**我该如何隐藏我tree view里的expander arrows？**

设置tree view这哦姑娘的expander-column属性可以隐藏列。参看`gtk_tree_view_set_expander_column()`和`gtk_tree_view_column_set_visible()`。
##### GTK+中使用cairo
###6.1
**我该如何在GTK+应用程序中使用cairo来绘制？**

"`draw`"信号接受一个准备使用的cairo上下文作为你应该使用的参数。
GTK+中所有的drawing 在draw处理函数中完成，GTK+为双缓冲drawing 创建了一个临时像素图。用`gtk_widget_set_double_buffered ()`可以容易的关闭双缓冲，但这并不合适，因为会造成一些闪烁。
###6.2
**我可以通过使用cairo中的Glitz或者GL来提升程序的表现么？**

不能。GDK X11后端使用cairo X 后端（其他GDK后端使用它们各自的原生cairo后端）。GTK+开发者相信优化cairo X 后端和使用的X server中的相关代码路径是提升GDK drawing表现最棒的方法（大部分是Render扩展）。
###6.3
**我可以用cairo在GdkPixbuf上绘制么？**

不能.至少现在不行。cairo image surface 不支持GdkPixbuf使用的像素格式。
