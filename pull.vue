<template>
  <div>

  </div>
</template>
<script>
export default {
  methods: {
    //代码中包含界面变化和数据更新，仔细看哦
    refreshTouchStart(e) {
      let touch = e.changedTouches[0];
      this.tipText = '下拉刷新';//下拉提示文字
      this.startY = touch.clientY;//获得当前按下点的纵坐标
    },

    refreshTouchMove(e) {
      this.$store.commit('bottomShowFalse');//与本逻辑无关，滑动时隐藏底部作用
      let touch = e.changedTouches[0];
      let _move = touch.clientY - this.startY;//获得滑动的距离
      this.bottomFlag = $('.present-box').offset().top + $('.present-box').height() - document.body.clientHeight <= 40;//滑动到底部标识
      if ($('.present-box').offset().top >= this.headerHeight) {//内容主体超出了一个头部的距离
        if (_move > 0 && _move < 1000) {//滑动距离>0代表下拉//<1000是为了防止神人无限拉阿拉
          this.el.style.marginTop = _move + 'px';//根据拉的距离，实现界面上的变化（界面变化）
          this.moveDistance = touch.clientY - this.startY;//记录滑动的距离，在松手后让他滑啊滑滑回去
          if (_move > 50) {//拉到一定程度再下拉刷新，防止误操作
            this.tipText = '松开即可刷新'//上面有了
          }
        }
      }
    },
    refreshTouchEnd() {
      this.$store.commit('bottomShowTrue');//松开后底部就biu的出现啦
      if (this.bottomFlag) {//若符合上拉加载的条件，则直接进行数据更新
        this.$emit('loadBottom');
      }
      let that = this;
      if (this.moveDistance > 50) {//拉了一定距离才触发加载动作
        this.tipText = '数据加载中...';
        let timer = setInterval(function () {
          that.el.style.marginTop = that.el.style.marginTop.split('px')[0] - 5 + 'px';//如果拉的很长，一次性弹回去影响用户体验，所以先让他弹到50的高度，然后再进行数据更新
          if (Number(that.el.style.marginTop.split('px')[0]) <= 50) {//小于50后就不进行界面变化了，先进行数据更新再变化
            clearInterval(timer);
            new Promise((resolve, reject) => {
              that.$emit('loadTop', resolve, reject);//通知父控件，下拉刷新条件满足了，你更新吧
            }).then(() => {
              that.resetBox();
            }).catch(() => {
              that.resetBox();//界面恢复（也就是弹回去啦）
            });
          }
        }, 1);//通过一个promise，让数据更新结束后再进行界面变化。也可以采用其他的方式，如async await方式
      } else {
        this.resetBox();
      }
    },
    resetBox() {
      let that = this;    //使用定时器的方式，biubiubiu的实现滑动界面刷新的效果。
      if (this.moveDistance > 0) {
        let timer = setInterval(function () {
          that.el.style.marginTop = that.el.style.marginTop.split('px')[0] - 1 + 'px';
          if (Number(that.el.style.marginTop.split('px')[0]) <= 0) clearInterval(timer);//这里很重要，不删除，可能看到奇奇怪怪的东西哦
        }, 1)
      }
      this.moveDistance = 0;
    }
  }
}
</script>
<style>

</style>