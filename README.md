# SOLID原则

## 什么是SOLID
> SOLID 是 Robert C. Martin（罗伯特·C·马丁-世界级编程大师）的前五个面向对象设计原则的首字母缩写词。这些原则的目的是：让你的代码、架构更具可读性、可维护性、灵活性。

## SOLID的五大原则
#### 1.单一职责原则(Single Responsibility Principle)
> 单一职责原则的定义是每个类应该只有一个职责，也就是只做一件事。在五大原则中，单一职责原则是最容易遵循的，也是最有影响力的一项，因为它极大提高了代码的质量。对于我们在项目开发中，我们可以尽可能将一个大的组件拆成多个职责功能单一的小组件，例如：
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
> 模块应该对扩展开放，而对修改关闭。开放封闭原则指出“一个软件实体（类、模块、函数）应该对扩展开放，对修改关闭”。开放封闭原则主张以一种允许在不更改源代码的情况下扩展组件的方式来构造组件。

下面来看一个场景，有一个可以在不同页面上使用的 Header 组件，根据所在页面的不同，Header 组件的 UI 应该有略微的不同
```
const Header = () => {
  const { pathname } = useRouter()
  return (
    <header>
      <Logo />
      <Actions>
        {pathname === '/dashboard' && <Link to="/create/user">create user</Link>}
        {pathname === '/' && <Link to="/dashboard">dashboard</Link>}
      </Actions>
    </header>
  )
}

const HomePage = () => (
  <>
    <Header />
    <OtherHomeStuff />
  </>
)

const DashboardPage = () => (
  <>
    <Header />
    <OtherDashboardStuff />
  </>
)
```

不难看出该组件的逻辑是根据所在页面的不同，呈现指向不同页面组件的链接。我们可以思考一下，如果需要将这个Header组件添加到更多的页面中会发生什么呢？每次创建新页面时，都需要引用 Header组件，并修改其内部实现。这种方式使得 Header 组件与使用它的上下文紧密耦合，并且违背了开放封闭原则。

为了解决这个问题，我们可以使用组件组合。Header组件不需要关心它将在内部渲染什么，相反，它可以将此责任委托给将使用 children 属性的组件：
```
const Header = ({ children }) => (
  <header>
    <Logo />
    <Actions>
      {children}
    </Actions>
  </header>
)

const HomePage = () => (
  <>
    <Header>
      <Link to="/create/user">create user</Link>
    </Header>
    <OtherHomeStuff />
  </>
)

const DashboardPage = () => (
  <>
    <Header>
      <Link to="/dashboard">dashboard</Link>
    </Header>
    <OtherDashboardStuff />
  </>
)
```
使用这种方法，我们完全删除了 Header 组件内部的变量逻辑。现在可以使用组合将想要的任何内容放在Header中，而无需修改组件本身。

遵循开放封闭原则，可以减少组件之间的耦合，使它们更具可扩展性和可重用性。

#### 3.里氏替换原则（Liskov Substitution Principle）
> 里氏替换原则可以理解为对象之间的一种关系，子类型对象应该可以替换为超类型对象。
> 通俗的定义就是：子类可以扩展父类的功能，但是不能改变父类原有的功能。



#### 4.接口隔离原则（ISP）
> 接口隔离原则主张最小化系统组件之间的依赖关系，使它们的耦合度降低，从而提高可重用性。

下面让我们来看这个例子
```
1、定义一个普通书本的接口对象
interface IOrdinarybookItem {
  id: number,
  name: string,
  price: string,
  thumb: string
}

2、定一个书本的数据接口
interface IBooks {
  data: IOrdinarybookItem[]
}

3、渲染书本的封面图
interface BookThumbProps {
  book: IOrdinarybookItem
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
上面例子中的BookThumb组件，我们不难发现它有一个问题，就是把整个书本的对象数据当作props传进去，实际上这个组件只用到book对象中的封面图thumb属性。

当我们需要需求有变动，需要展示别的类型的缩略图，但是它的属性不叫thumb，下面我们来改造一下。

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
> 依赖倒置原则指出“要依赖于抽象，不要依赖于具体”。换句话说，一个组件不应该直接依赖于另一个组件，而是它们都应该依赖于一些共同的抽象。这里“组件”是指应用程序的任何部分，可以是 React 组件、函数、模块或第三方库。

看下面的一个例子
```
import postUserInfo from '@/api/postUserInfo'
const UserForm = () => {
  const [form] = Form.useForm()
  const onFinish = useCallback((data) => {
    postUserInfo(data).then(res => {
      console.log(res)
    })
  },[])
  return (
    <Modal
      title='更新用户信息'
      visible={true}
      okButtonProps={{
        htmlType: 'submit',
        onClick: () => {
          form.submit()
        },
      }}
    >
      <Form form={form} onFinish={onFinish}>
        <Form.Item
          label="用户名"
          name={'username'}
        >
          <Input placeholder="请输入用户名" />
        </Form.Item>
      </Form>
    </Modal>
  )
}
```
这段代码主要是一个更新用户名称的弹窗组件，组件直接依赖了更新用户名称的api接口，因此它们之间存在紧密耦合，这种依赖关系就会导致一个组件的更改会影响其他组件。

依赖倒置原则就提倡打破这种耦合，对于这种我们可以直接把接口抽离出来，提交数据的逻辑可以通过onFinish提交的回调抽象出来，由父组件负责提供该逻辑的具体实现。

```
1、在子组件中提交把表单数据传递给父组件做提交逻辑处理
import postUserInfo from '@/api/postUserInfo'
interface UserFormProps {
  onSubmit: (data) => void
}
const UserForm = ({ onSubmit }: UserFormProps) => {
  const [form] = Form.useForm()
  const onFinish = useCallback((data) => {
    onSubmit(data)
  },[])
  return (
    <Modal
      title='更新用户信息'
      visible={true}
      okButtonProps={{
        htmlType: 'submit',
        onClick: () => {
          form.submit()
        },
      }}
    >
      <Form form={form} onFinish={onFinish}>
        <Form.Item
          label="用户名"
          name={'username'}
        >
          <Input placeholder="请输入用户名" />
        </Form.Item>
      </Form>
    </Modal>
  )
}

2、在父组件中处理接口逻辑
import postUserInfo from '@/api/postUserInfo'
const ModalUserForm = () => {
  const handleSubmit = useCallback((data) => {
    postUserInfo(data).then(res => {
      console.log(res)
    })
  },[])
  return (
    <UserForm onSubmit={handleSubmit} />
  )
}
```
ModalUserForm组件充当api和UserForm之间的粘合剂，而它们本身保持完全独立。这样就可以对这两个组件进行单独的修改和维护，而不必担心修改会影响其他组件。

依赖倒置原则旨在最小化应用程序不同组件之间的耦合，最小化是所有SOLID原则中反复出现的关键词——从最小化单个组件的职责范围到最小化它们之间的依赖关系等等。
