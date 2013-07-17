**Golang 类型合法的转换** *float64 同 float32 类似*  

|**+From**|xb []byte|xi []int|xr []rune|s string|f float32|i int|  
|---|---|---|---|---|---|---|
|**+To** |  
|+ []byte|×|||[]byte(s)|
|+ []int||×||[]int(s)|
|+ []rune|||×|[]rune(s)|
|+ string|string(xb)|string(xi)|string(xr)|x|
|+ float32|||||x|float32(i)|
|+int|||||int(f)|x|
