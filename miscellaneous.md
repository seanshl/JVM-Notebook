## 杂项

* **声明class为final对于性能的影响**

  virtual (overridden) methods generally are implemented via some sort of table (vtable) that is ultimately a function pointer. Each method call has the overhead of having to go through that pointer. When classes are marked final then all of the methods cannot be overridden and the use of a table is not needed anymore - this it is faster.
  Some VMs (like HotSpot) may do things more intelligently and know when methods are/are not overridden and generate faster code as appropriate.
