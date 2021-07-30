# html2canvas 使用

```
import html2canvas from "html2canvas";
.
.
.
// 回到顶部, 防止部分截图出现空白不全
window.pageYOffset = 0
document.documentElement.scrollTop = 0
document.body.scrollTop = 0
// 部分安卓上还是无法回滚到顶部 app是要截图的容器最顶部
document.querySelector('.app').scrollIntoView()

let dom = this.$refs.imageTofile
const width = dom.offsetWidth
const height = dom.offsetHeight //包裹容器的高度
html2canvas(dom, {
  scrollY: 0,
  background: 'transparent', //  透明
  width: width, //画布的宽度，即生成图片的宽度
  height: height, //画布的宽度，同上
  scale: 3, // 缩放
  useCORS: true, // 是否使用图片跨域，如何使本地图片不用配置
  allowTaint: true, //允许污染
}).then((canvas) => {
  let url = canvas.toDataURL("image/png");
  //TODO : url do something
});

```
