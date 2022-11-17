tags:: Effect Api, Vue源码
page-type:: [[api]]

- ## test case
- ```typescript
  it('stop', () => {
    let dummy;
    const obj = reactive({ prop: 1 });
    const runner = effect(() => {
      dummy = obj.prop;
    });
    obj.prop = 2;
    expect(dummy).toBe(2);
    stop(runner);
    obj.prop++;
    expect(dummy).toBe(2);
  
    // stopped effect should still be manually callable
    runner();
    expect(dummy).toBe(3);
  });
  ```
- > 这里`obj.props++`可以等价于`obj.props = obj.props + 1`
  > 在赋值语句中，先触发了obj的get方法重新收集依赖，所以之前stop函数中删除的effect又被收集回来了
- 解决方法是设置一个`shouldTrack`的全局标志，执行依赖后将标志置为false，若标志处于false则在响应式对象触发get方法时跳过依赖收集
	- commit diff：[https://github.com/pidanmeng/mini-vue/commit/68a39345c0e0fde674fd07fd1169741929a96682](https://github.com/pidanmeng/mini-vue/commit/68a39345c0e0fde674fd07fd1169741929a96682)
- ## #A
- Question：以下代码也能通过测试case：
	- ```typescript
	  run() {
	    try {
	      activeEffect = this;
	      shouldTrack = true;
	    } catch (e) {
	      console.warn(e);
	    }
	    const result = this.fn();
	    // 把activeEffect置为空
	    activeEffect = undefined;
	    return result;
	  }
	  ```
	- 为什么不用activeEffect = undefined呢？
- Answer：
	- 一个`Effect`可能被多个响应式对象依赖，如果这样处理会导致只有第一个被触发`get`方法的响应式对象成功收集依赖，其他响应式对象依赖收集失败的bug