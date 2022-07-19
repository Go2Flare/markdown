# 内存对齐

- go对结构的底层优化，能增加CPU读取的性能

```go
func TestIntUintByte(){
	type myStruct struct{
		A uint8 //1个字节
		B uint16 //2
		C uint32 //4

		D int8 //1
		E int16 //2
		F int32 //4

		G byte //1
	}

	//1. 案例
	//01 02 00 03 00 00 00 04  05 00 06 00 00 00 07
	mst := myStruct{A:1, B:2, C:3, D:4, E:5,F:6,G:7}
	fmt.Println(unsafe.Alignof(mst))
	fmt.Println(unsafe.Sizeof(mst))
	fmt.Println(hex.Dump((*(*[20]byte)(unsafe.Pointer(&mst)))[:]))

	//2. 分析
	//有3个结果，15，28，20
	//是按照4字节对齐，最后才是20
	buffer := bytes.NewBuffer(nil)
	binary.Write(buffer, binary.LittleEndian, &mst)
	fmt.Println(hex.Dump(buffer.Bytes()))

	var mst1 myStruct
	binary.Read(buffer, binary.LittleEndian, &mst1)
	fmt.Println(hex.Dump((*(*[20]byte)(unsafe.Pointer(&mst1)))[:]))
}
```

## 类型转换

- 转换技巧，将数组转切片

将数组取指针，在使用()[:]格式进行转换

```go
fmt.Println(reflect.TypeOf(*(*[20]byte)(unsafe.Pointer(&mst1))))
fmt.Println(reflect.TypeOf((*(*[20]byte)(unsafe.Pointer(&mst1)))[:]))
fmt.Println((&[3]byte{1,2,3})[:])
//[20]uint8
//[]uint8
//[1 2 3]
```

- 强制类型转换

```go
type t struct{A string;B []string}
a := t{A:"1",B:[]string{"1","2","3"}}
type t1 struct{C string;D []string}
fmt.Println(*(*t1)(unsafe.Pointer(&a)))
```

