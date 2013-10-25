Markdown的语法是为了一个目的：用来编写web的一种格式。

Markdown不是HTML的替代品，也不像它。他的语法很少，仅对应于HTML的标签的子集。其重点不是制定一些更便捷的插入HTML标签的语法。在我看来，HTML标签已经很容易插入了。Markdown的重点在于更容易的读，写和编辑文章。

HTML是一种发布格式；Markdown是一种编写格式。

这样来看，Markdown的语法仅用来解决可以用普通文本来传播的话题。

对于任何不能用Markdown语法来解决的标记，你就用HTML。不需要事先说你已经从Markdown转向了HTML，或者划清界限；你就是在使用标签罢了。

唯有一些限制是一些块级的HTML元素 — 例如 `<div>`, `<table>`, `<pre>`, `<p>` 等 

-必须和周围的内容以 *空行分开*，另外块标签的开始和结束 *不能用空行或者tab来缩进*。 Markdown足够聪明不会在HTML的块标签周围添加额外的（不想要的）`<p>`标签。

比如，在Markdown的文章里添加一个HTML表格：

This is a regular paragraph.

	<table>
	    <tr>
	        <td>Foo</td>
	    </tr>
	</table>

This is another regular paragraph.

注意观察Markdown的格式语法在HTML的块标签里并没有生效。 比如， 你不能在HTML块里添加 *强调*, 像`<span>`, `<cite>`, or `<del>` 的span级的HTML标签可以在Markdown的任何段落， 列表或者标题内使用。 如果你乐意， 你可以不用Markdown而使用HTML标签， 例如相比markdown你可能更愿意使用HTML的 `<a>` 或者 `<img>` 标签， 这样完全可以。

不像块级标签， Markdown的语法可以在span级标签内处理。