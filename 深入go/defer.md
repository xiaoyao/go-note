# defer

defer是在return之前执行的。这个在 官方文档中是明确说明了的。

要使用defer时不踩坑，最重要的一点就是要明白，return xxx这一条语句并不是一条原子指令!