---
title: vuedraggable 拖拽异步组件问题记录
date: 2018-07-30 10:46:51
tags: [vue,vuedraggable,Sortablejs,vue异步组件]
---

最近一直在做[可视化拖拽系统](https://github.com/ithack/vueCMS)，一期基本功能已经完成，二期在完善系统，第一步就是把所有组件异步加载，这样有些组件不用的话不必要加载，js也就没那么大了，本来认为很简单一件事，但当异步组件遇上vuedrag就出了问题；研究了很长一段时间vuedraggable的实现方式，最终找到了问题，首先说一下问题

我们可视化系统拖拽组件，当把组件异步处理后会出现连续拖拽两个相同的异步组件会出现store里的JSON Tree多出一个非DOM节点，这个DOM节点是在component组件名进入前已经把左边组件DOM放到了这个JSON Tree里，这样就出现DOM树结构解析出现问题，我查了一下vuedraggable的API，发现并没有任何一个执行顺序可以控制add的插入节点字符串，这样就导致了后边再拖拽就会导致component解析问题；

在查找代码后发现，其实在draggable拖拽松手那一刻到add回到那一刻分别由vue和sortablejs执行了两个事件，一个是vue的input事件（也就是v-model用到的数据绑定）另一个是sortable里的change事件源码如下：

<!--more-->

```
……
        emitChanges: function emitChanges(evt) {
          var _this5 = this;

          this.$nextTick(function () {
            _this5.$emit('change', evt);
          });
        },
        alterList: function alterList(onList) {
          if (!!this.list) {
            onList(this.list);
          } else {
            var newList = [].concat(_toConsumableArray(this.value));
            onList(newList);
            this.$emit('input', newList);
          }
……
        spliceList: function spliceList() {
          var _arguments = arguments;

          var spliceList = function spliceList(list) {
            return list.splice.apply(list, _arguments);
          };
          this.alterList(spliceList);
        },
        updatePosition: function updatePosition(oldIndex, newIndex) {
          var updatePosition = function updatePosition(list) {
            return list.splice(newIndex, 0, list.splice(oldIndex, 1)[0]);
          };
          this.alterList(updatePosition);
……
        onDragAdd: function onDragAdd(evt) {
          var element = evt.item._underlying_vm_;
          if (element === undefined) {
            return;
          }
          removeNode(evt.item);
          var newIndex = this.getVmIndex(evt.newIndex);
          this.spliceList(newIndex, 0, element);
          this.computeIndexes();
          var added = { element: element, newIndex: newIndex };
          this.emitChanges({ added: added });
        },
        onDragRemove: function onDragRemove(evt) {
          insertNodeAt(this.rootContainer, evt.item, evt.oldIndex);
          if (this.isCloning) {
            removeNode(evt.clone);
            return;
          }
          var oldIndex = this.context.index;
          this.spliceList(oldIndex, 1);
          var removed = { element: this.context.element, oldIndex: oldIndex };
          this.resetTransitionData(oldIndex);
          this.emitChanges({ removed: removed });
        },
……

```

上边代码名字可以看出在左边组件拖到可视区域松手那一刻，先出发了vue的input事件回调传参事事左侧的dom节点，而且这个newIndex的索引记录的事上一次松手时的位置，这就到这了第一次 从手我们json进入JSON Tree的第0个节点，而第二次json进入了上一次松手位置的索引节点位置，而在执行add添加真正组件的时候就导致了JSON Tree出现问题，如图：![2](/Users/yangkai9/Desktop/project/ithack/source/images/vuedraggable拖拽异步组件问题记录/2.gif)

通过vue开发工具我们可看到事件触发顺序input-->change-->add,通过log我们可以在三个事件中分别查看newIndex的索引数，我们第一次都是0，但第二次的时候input是上一次的拖拽位置索引，而从change开始就是当前索引，也就是结合代码可发现，其实在我们change之前input已经把未配置的JSON更新到JSON Tree里了，这样就会出现问题；

好了问题找到了，下面解决问题就好说了，我们在input更新了JSON Tree，我们可以在add之前也就是change事件里记录一下input的的newIndex索引，然后在add回调事直接删掉这个没用的索引就可以了；代码如下：

```
……
	oninput(s){
      s.map((item,index)=>{
        if(item.placeholder){
          console.log(item,index)
        }
      })
    },
    onchange(a){
      this.old=a.added.newIndex
    },
    onAdd (sort) {
      console.log(this.old)
      let { item, newIndex}=sort;
      this.node.children.splice(this.old,1)
      ……
    }
……
```

这样就解决了异步组件+vuedraggable的问题，当然有人会想同步也会有问题啊，这里解释一下同步确实也有这个问题，但我们的只需要在add回调里用splice(newIndex,1,新的节点json)直接替换就可以解决问题，组件异步后由于input会比组件新的节点JSON进入的早，但不一定是在newIndex这个索引下，所以如果用splice直接修改newIndex这时候就不灵了；

可能其他的vue拖拽组件异步后不会出现问题，因为结构和需求不一样，这里只作为类似问题和需求的防坑指南记录，欢迎吐槽！

