# SOLID原则

## 什么是SOLID
> SOLID 是 Robert C. Martin（世界级编程大师）的前五个面向对象设计原则的首字母缩写词。这些原则的目的是：让你的代码、架构更具可读性、可维护性、灵活性。

## SOLID的五大原则
#### 1.单一职责原则(Single Responsibility Principle)
> 单一职责原则的定义是每个类应该只有一个职责，也就是只做一件事。
> 在五大原则中，单一职责原则是最容易遵循的，也是最有影响力的一项，因为它极大提高了代码的质量。
> 对于我们在项目开发中，我们可以尽可能将一个大的组件拆成多个职责功能单一的小组件，例如：
> * 将功能较多的大型组件拆分为较小的组件；
> * 将与组件功能无关的代码提取到单独的函数中；
> * 将有联系的功能提取到自定义Hooks中。

例如下面是一个显示已上架的商品列表组件
```
const ShelvesProductList = () => { 
  const [products, setProducts] = useState([])
  useEffect(() => {
    const getProducts = async () => { 
      const response = await fetch('/api/getProduct') 
      const data = await response.json() 
      setProducts(data)
   }
   getProducts() 
  }, [])
  
  return ( 
    <ul> {
      products.filter(item => item.isShelves).map(item => 
       <li key={item.id}>
        <img src={item.pic} />
        <p>{item.productName}</p>
        <span>{item.productPrice}</span>
       </li>
     )}
    </ul>
  ) 
}
```
不难发现这个组件一共做了三件事情，获取数据、过滤数据、渲染数据，那我们该如何拆分呢？

```
1、获取所有商品数据自定义hook
const useProducts = () => {
  const [products, setProducts] = useState([])
  useEffect(() => {
    const getProducts = async () => {
      const response = await fetch('/api/getProduct') 
      const data = await response.json() 
      setProducts(data)
    }
    getProducts()
  }, [])
  return { products }
}

2、过滤符合条件的商品数据
const getShelvesProducts = (products) => {
  return products.filter(item =>  item.isShelves)
}

3、只获取符合条件的商品数据自定义hook
const useShelvesProducts = () => {
  const { products } = useProducts()
  const shelvesProducts = useMemo(() => getShelvesProducts(products), [products])
  return { shelvesProducts }
}

4、单个数据的渲染
const ProductItem = ({product}) => {
  return (
    <li>
      <img src={product.pic} />
      <p>{product.productName}</p>
      <span>{product.productPrice}</span>
    </li>
  )
}

5、最终渲染数据
const ShelvesProductList = () => { 
  const { shelvesProducts } = useShelvesProducts()
  return ( 
    <ul> 
      {shelvesProducts.map(item => 
        <ProductItem key={item.id} product={item} />
      )}
    </ul>
  ) 
}

```

#### 2.开闭原则（Open-Closed Principle）
> 模块应该对扩展开放，而对修改关闭
> 开放封闭原则指出“一个软件实体（类、模块、函数）应该对扩展开放，对修改关闭”。开放封闭原则主张以一种允许在不更改源代码的情况下扩展组件的方式来构造组件。


#### 3.里氏替换原则（Liskov Substitution Principle）
> 里氏替换原则可以理解为对象之间的一种关系，子类型对象应该可以替换为超类型对象。

#### 4.接口隔离原则（ISP）
> 接口隔离原则主张最小化系统组件之间的依赖关系，使它们的耦合度降低，从而提高可重用性。

下面让我们来看这个例子
```
1、定义一个书本的接口对象
interface IBookItem {
  id: number,
  name: string,
  price: string,
  thumb: string
}

2、定一个书本的数据接口
interface IBooks {
  data: IBookItem[]
}

3、渲染书本的封面图
interface BookThumbProps {
  book: IBookItem
}
const BookThumb = ({ book }: BookThumbProps) => {
  return <img src={book.thumb} alt="alt" />
}

4、渲染书本数据
const Books = ({data}: IBooks) => {
  return (
    <ul>
      {data.map(item => <BookThumb key={item.id} book={item} />)}
    </ul>
  )
}
```
上面例子中的BookThumb组件，我们不难发现它有一个问题，就是把整个书本的对象数据当作props传进去，实际上这个组件只用到book对象中的封面图thumb属性。当我们需要需求有变动，需要展示别的类型的缩略图，但是它的属性不叫thumb，下面我们来改造一下。

```
1、定义一个普通书本的接口对象
interface IOrdinarybookItem {
  id: number,
  name: string,
  thumb: string
}

2、教科书类型的书包接口对象
interface ITextbookItem {
  id: number,
  name: string,
  textThumb: string
}

3、定一个书本的数据接口
interface IBooks {
  data: IOrdinarybookItem[] | ITextbookItem[]
}

4、改造一下，这里只传递显示缩略图的属性（并非整个书本对象）
interface BookThumbProps {
  thumbUrl: string
}

const BookThumb = ({ thumbUrl }: BookThumbProps) => {
  return <img src={thumbUrl} alt="alt" />
}

5、渲染书本数据
const Books = ({data}: IBooks) => {
  return (
    <ul>
      {data.map(item => <BookThumb key={item.id} thumbUrl={'thumb' in item ? item.thumb : item.textThumb} />)}
    </ul>
  )
}
```

#### 5.依赖倒置原则（DIP）
> 程序要依赖于抽象接口，不要依赖于具体实现。简单的说就是要求对抽象进行编程，不要对实现进行编程，这样就降低了客户与实现模块间的耦合。
