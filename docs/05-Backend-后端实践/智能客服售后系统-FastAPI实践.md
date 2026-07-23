# Day1_4_08: 智能客服售后系统

##  代码管理工具

### 

## 要点

### 1.不能用print，用logging输出日志

### 2. 软件包的__init__文件里可以直接引用公共对象，同为包的其它代码文件可以直接用过from . import xx来引用



## 登录注册

### jwt令牌和Oauth2.0协议的登录注册



### 1. jwt令牌，用户登录LoginResponse...（数据库数据）...的结构体

### 2. 密码对比，明文与哈希值对比（调库）



### Session 的核心作用：

1. 数据库操作入口 ：所有 CRUD 操作都通过 Session 执行
2. 连接管理 ：自动管理数据库连接的创建和释放
3. 对象状态跟踪 ：跟踪对象的变更，自动生成 SQL
4. 事务支持 ：提供事务管理（commit/rollback）
5. 资源管理 ：确保数据库连接正确关闭



### tenants接口数据流(注册页面下拉列表获取企业信息)：

前端请求—> GET /api/v1/auth/tenants

后端路由—>@router.get("/tenants")

read_tenants(db:Session = Depends(databases.get_db),response_model=List[schemas.TenantOut])

Depends 告诉 FastAPI："这个参数需要由某个函数来提供，请自动调用那个函数并把结果传给我"

连接数据库 Session = Depends(databases.get_db)

```python
engine = create_engine(DATABASE_URL,echo=False)
SessionLocal = sessionmaker(autocommit=False,autoflush=False,bind=engine)
def get_db():
    db = SessionLocal()
    try:
        yield db # yieled这个步骤暂时暂停，该步产生的对象可以先拿出去用，对象被接收方用完了回来接着跑流程
    finally:
    	db.close()
```

```python
def read_tenants(db: Session = Depends(databases.get_db)):
	# ORM对象列表List[ TenantDB{id，name，created_at} ]
    tenants = db.query(base.TenantDB).all()
    # response_model=List[schemas.TenantOut] 路由接口要求你返回这个模型定义的数据格式(TenantDB里面可能会有敏感数据，TenantOut不定义，防止泄露敏感数据)
    return tenants
- db.query(base.TenantDB) ：创建一个查询 tenants 表的查询对象
- .all() ：执行查询并返回所有结果
- 返回类型 ： List[TenantOUT] （TenantOUT ORM 对象的列表）
- 最终响应 ：FastAPI 自动将 ORM 对象转换为 JSON 格式返回给前端
```

返回了tenants的json消息，前端获取name字段放入下滑列表



### register接口数据流(注册功能)：

用户填写表单

​    ↓
用户名：zhangsan
密码：123456
选择公司：腾讯 (tenant_id=2)
​    ↓
前端发送 POST 请求
POST /api/v1/auth/register
{
  "username": "zhangsan",
  "password": "123456",
  "tenant_id": 2
}
​    ↓
register端口接收并验证

@router.post("/register", response_model=schemas.UserOut, status_code=status.HTTP_201_CREATED)

```
def register(

  user_in: schemas.UserCreate, 

  db: Session = Depends(database.get_db)
):
  \# 1. 检查用户名是否已存在
  user = crud.get_user_by_username(db, username=user_in.username)
  if user:

      raise HTTPException(

      status_code=400,

      detail="用户名已存在",
   )
  \# 2. 提取 tenant_id
  \# 如果前端没传，这里默认为 1（或者你可以抛出异常提示必须选择租户）
  if not user_in.tenant_id:
      raise HTTPException(status_code=400, detail="必须选择所属租户")
  \# 3. 调用 CRUD 创建用户
  \# 修改点：显式传入 tenant_id 参数
  user = crud.create_user(db, user_in=user_in, tenant_id=user_in.tenant_id)
 
  return user
```



1. 检查用户名是否存在 → 不存在 ✅

2. 检查 tenant_id → 2 ✅
    ↓
CRUD 层处理
    
    def create_user(db: Session, user_in: schemas.UserCreate, tenant_id: int):
    
      hashed_password = security.get_password_hash(user_in.password) #加密
    
3. 密码加密 → bcrypt.hash("123456")

4. 创建用户对象(base.UserDB)
   {
     username: "zhangsan",
     password: "$2b$12$KIXxh...加密后的哈希...",
     tenant_id: 2,
     role: 0
   }

5. 保存到数据库
    ↓
    返回用户信息（不含密码）(shcemas.UserOut)
    {
      "id": 5,
      "username": "zhangsan",
      "tenant_id": 2,
      "nickname": null,
      "created_at": "2024-01-01T12:00:00"
    }
    ↓
    前端显示"注册成功"，跳转到登录页



### login登录接口数据流:

用户输入
用户名：zhangsan
密码：123456
    ↓
前端发送 POST 请求
POST /api/v1/auth/login
Content-Type: multipart/form-data
username=zhangsan&password=123456
    ↓
后端接收表单数据
OAuth2PasswordRequestForm=Depends() (解析表单数据)
form_data.username = "zhangsan"
form_data.password = "123456"
    ↓
查询数据库
SELECT * FROM users WHERE username = 'zhangsan'
    ↓
找到用户
{
  id: 5,
  username: "zhangsan",
  password: "$2b$12$KIXxh...加密后的哈希...",
  tenant_id: 2,
  role: 0
}
    ↓
验证密码
security.verify_password("123456", "$2b$12$KIXxh...")
    ↓
密码正确 ✅
    ↓
生成 JWT Token
{
  "exp": 1704067200,  // 30 分钟后过期
  "sub": "5"          // 用户 ID
}
encoded_jwt = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
    ↓
返回响应
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "user": {
    "id": 5,
    "username": "zhangsan",
    "tenant_id": 2,
    "role": 0,
    "nickname": null
  }
}
    ↓
前端保存 Token 和用户信息
localStorage.setItem('token', 'eyJhbGci...')
localStorage.setItem('user', JSON.stringify({...}))
    ↓
跳转到首页

