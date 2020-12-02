> 作业
我们在数据库操作的时候，比如 dao 层中当遇到一个 sql.ErrNoRows 的时候，是否应该 Wrap 这个 error，抛给上层。为什么？应该怎么做请写出代码

> 分析
```
举个栗子：
GET {host}/product/:product_id 如果找不到该ID，应该报错
总结：
    一般场景: 来说，dao层应该报错，然后在service层的上游handle将错误转化为面向web端 错误Code以及用户可读的错误TipText
    二般场景: 如果我们的需求方要求后端吞掉99%的这种报错，或者降级，那么应该在该业务的service层内消化或者掉降级服务
```

> 伪代码：
- DAO 层
```go
type Dao struct {}

var (
	ErrProductNotFound = errors.New("product not found")
	......
)

func New() *Dao {
	return &Dao{}
}

func (d *Dao) FindProductByID(ProductID int) (product *model.Product, err error) {
	err = DB.Table("product").Where("id = ?", ProductID).Find(product).Error
	if errors.Is(err, sql.ErrNoRows) {
		err = ErrProductNotFound
	}
	//......
	if err != nil {
		err = errors.Wrap(err, fmt.Sprintf("find by product_id: %v error", ProductID))
	}
	return
}
```

- Service 层
```go
type Service struct {}

......
// 一般场景
func (s *Service) FindProductByID(ProductID int) (*model.Product, error) {
    return dao.FindProductByID(ProductID)
}

// 二般场景
func (s *Service) MustFindProductByID(ProductID int) (product *model.Product, err error) {
	product, err = dao.FindProductByID(ProductID)
	if errors.Is(err, dao.ErrProductNotFound) {
		product = dao.GetMockProduct()
		err = nil
	}
	return
}
```
