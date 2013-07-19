# 空接口
由于每个类型都能匹配到空接口: interface{}。我们可以创建一个接受空接口 作为参数的普通函数:  

Listing 6.2. 用空接口作为参数的函数  

	func g(something interface{}) int {
	 	return something.(I).Get()
	}  

在这个函数中的 return something.(I).Get() 是有一点窍门的。  
值` something `具有类型 interface{},这意味着方法没有任何约束:它能包含任何类型。  
` .(I) `是类型断言,用于转换 something 到 I 类型的接口。  
如果有这个类型,则可以调用` Get() `函数。因此,如果创建一个 *S 类型的新变量,也可以调用 g(),因为 `*S` 同样实现了空接口。  

	s = new(S) 
	fmt.Println(g(s));  

调用 g 的运行不会出问题,并且将打印 0。如果调用 g() 的参数没有实现 I 会带来一个麻烦:  

Listing 6.3. 实现接口失败   
		
		i:=5 ←声明i 是一个``该死的''int
        fmt.Println(g(i))
这能编译,但是当运行的时候会得到:
panic: interface conversion: int is not main.I: missing method Get 这是绝对没问题,内建类型 int 没有 `Get()` 方法。
