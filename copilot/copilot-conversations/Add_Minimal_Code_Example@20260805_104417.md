---
epoch: 1785897857972
modelKey: "deepseek-v4-flash|deepseek"
topic: "Add Minimal Code Example"



tags:
  - copilot-conversation
---

**user**: 根据这段笔记，补充上一个极简的代码示例
[Timestamp: 2026/08/05 10:44:17]

**ai**: 可以在笔记中补充一个极简的 JavaScript 示例，直接覆盖你写的四个要点：

```javascript
// 核心：事件名 = 回调函数数组
class EventBus {
  constructor() {
    this.events = {}; // { 事件名: [回调函数, ...] }
  }

  // 接收方：提前注册监听（对应第 3 点）
  on(event, callback) {
    (this.events[event] ||= []).push(callback);
  }

  // 发送方：广播（对应第 2 点）
  emit(event, ...args) {
    (this.events[event] || []).forEach(cb => cb(...args));
  }

  // 随时取消订阅（对应"典型特征"）
  off(event, callback) {
    this.events[event] = (this.events[event] || []).filter(cb => cb !== callback);
  }
}

// ---- 使用示例 ----

const bus = new EventBus();

// 组件 A（接收方）
bus.on('userLogin', (user) => {
  console.log('欢迎,', user.name);
});

// 组件 B（接收方），可随时取消
const onLogout = () => console.log('已退出');
bus.on('userLogout', onLogout);
bus.off('userLogout', onLogout); // 取消后不再触发

// 组件 C（发送方），完全不知道谁在监听
bus.emit('userLogin', { name: '张三' });
```

---

对应关系：

- `this.events` ↔ 第 1 点的「映射表」
- `emit` ↔ 第 2 点的「广播」
- `on` ↔ 第 3 点的「注册监听」
- `off` ↔ 「随时订阅 / 取消订阅」

配合使用示例，`bus.emit` 时 A、B 都能收到，但 C 并不需要知道它们的存在，这样就体现了第 4 点的「解耦」。
[Timestamp: 2026/08/05 10:44:25]