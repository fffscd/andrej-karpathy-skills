# 示例

[English](./EXAMPLES.md) | 简体中文

这些是真实编码场景示例，用来展示四个原则。每个示例都会说明 LLM 常见错误，以及更合适的处理方式。

---

## 1. 编码前思考

### 示例 1：隐藏假设

**用户请求：** “添加一个导出用户数据的功能”

**❌ LLM 常见做法（错误假设）**

```python
def export_users(format='json'):
    """将所有用户导出到文件。"""
    users = User.query.all()
    
    if format == 'json':
        with open('users.json', 'w') as f:
            json.dump([u.to_dict() for u in users], f)
    elif format == 'csv':
        with open('users.csv', 'w') as f:
            writer = csv.DictWriter(f, fieldnames=['id', 'email', 'name'])
            writer.writeheader()
            writer.writerows([u.to_dict() for u in users])
    
    return f"已导出 {len(users)} 个用户"
```

**问题：**
- 假设应该导出所有用户（分页和隐私怎么办？）
- 在没有询问的情况下假设文件位置
- 假设要包含哪些字段
- 没有检查真实数据结构就假设 CSV 字段名

**✅ 更合适的处理方式（说明假设）**

```
实现前，我需要先澄清：

1. **范围**：导出所有用户，还是筛选后的子集？（涉及隐私）
2. **格式**：你说的“导出”可能指：
   - 浏览器下载文件？
   - 后台任务完成后发送邮件通知？
   - 返回数据的 API 端点？
3. **字段**：要包含哪些用户字段？（有些字段可能敏感）
4. **数据量**：通常有多少用户？（会影响实现方案）

最简单的方案：添加一个返回分页 JSON 的 API 端点。
如果要做文件导出，需要更多信息。

你倾向哪种方式？
```

### 示例 2：多种理解

**用户请求：** “让搜索更快”

**❌ LLM 常见做法（默默选择一种解释）**

```python
# 添加缓存、数据库索引和异步处理，
# 没有先确认“更快”具体指什么
@lru_cache(maxsize=1000)
async def search(query: str) -> List[Result]:
    # ... 200 行优化代码
```

**✅ 更合适的处理方式**

```
“让搜索更快”可能有几种含义：

1. **响应时间更短**（从 500ms 降到 100ms 以内）
   - 添加数据库索引
   - 缓存高频查询
   - 预估耗时：2 小时

2. **支持更多并发搜索**（吞吐量）
   - 添加 async/await
   - 使用连接池
   - 预估耗时：4 小时

3. **体感速度更快**（用户体验）
   - 先展示部分结果
   - 渐进式加载
   - 预估耗时：3 小时

当前典型查询耗时约 500ms。
你的场景最关注哪一项？
```

---

## 2. 简洁优先

### 示例 1：过度抽象

**用户请求：** “添加一个计算折扣的函数”

**❌ LLM 常见做法（过度设计）**

```python
from abc import ABC, abstractmethod
from enum import Enum
from typing import Protocol, Union
from dataclasses import dataclass

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, amount: float) -> float:
        pass

class PercentageDiscount(DiscountStrategy):
    def __init__(self, percentage: float):
        self.percentage = percentage
    
    def calculate(self, amount: float) -> float:
        return amount * (self.percentage / 100)

class FixedDiscount(DiscountStrategy):
    def __init__(self, fixed_amount: float):
        self.fixed_amount = fixed_amount
    
    def calculate(self, amount: float) -> float:
        return min(self.fixed_amount, amount)

@dataclass
class DiscountConfig:
    strategy: DiscountStrategy
    min_purchase: float = 0.0
    max_discount: float = float('inf')

class DiscountCalculator:
    def __init__(self, config: DiscountConfig):
        self.config = config
    
    def apply_discount(self, amount: float) -> float:
        if amount < self.config.min_purchase:
            return 0
        discount = self.config.strategy.calculate(amount)
        return min(discount, self.config.max_discount)

# 只是做一次简单计算，却需要 30 多行配置
```

**✅ 更合适的处理方式（简单）**

```python
def calculate_discount(amount: float, percent: float) -> float:
    """计算折扣金额。percent 应为 0-100。"""
    return amount * (percent / 100)

# 用法
discount = calculate_discount(100.0, 10.0)  # 优惠 10 美元
```

**何时增加复杂度：** 只有在确实需要多种折扣类型时再增加。如果后续出现该需求，再重构。

### 示例 2：推测性功能

**用户请求：** “把用户偏好保存到数据库”

**❌ LLM 常见做法（添加了不必要功能）**

```python
class PreferenceManager:
    def __init__(self, db, cache=None, validator=None):
        self.db = db
        self.cache = cache or InMemoryCache()
        self.validator = validator or DefaultValidator()
    
    def save(self, user_id: int, prefs: dict, 
             merge: bool = True, 
             validate: bool = True,
             notify: bool = False) -> bool:
        """保存偏好，带可选合并、验证和通知。"""
        
        if validate:
            errors = self.validator.validate(prefs)
            if errors:
                raise ValidationError(errors)
        
        existing = self.get(user_id) if merge else {}
        
        if merge:
            existing.update(prefs)
            prefs = existing
        
        self.db.execute(
            "UPDATE users SET preferences = ? WHERE id = ?",
            (json.dumps(prefs), user_id)
        )
        
        if self.cache:
            self.cache.set(f"prefs:{user_id}", prefs)
        
        if notify:
            self.notify_preference_change(user_id, prefs)
        
        return True
    
    def notify_preference_change(self, user_id: int, prefs: dict):
        # 又为一个没人要求的功能写了 30 行
        pass
```

**✅ 更合适的处理方式（只做用户要求的事）**

```python
def save_preferences(db, user_id: int, preferences: dict):
    """将用户偏好保存到数据库。"""
    db.execute(
        "UPDATE users SET preferences = ? WHERE id = ?",
        (json.dumps(preferences), user_id)
    )
```

**以后需要时再添加：** 缓存（当性能成为问题时）、验证（当出现错误数据时）、合并（当需求出现时）。

---

## 3. 精准修改

### 示例 1：顺手重构

**用户请求：** “修复空邮箱会让验证器崩溃的 bug”

**❌ LLM 常见做法（改得太多）**

```diff
  def validate_user(user_data):
-     # Check email format
-     if not user_data.get('email'):
+     """验证用户数据。"""
+     email = user_data.get('email', '').strip()
+     
+     # 验证邮箱
+     if not email:
          raise ValueError("Email required")
-     
-     # Basic email validation
-     if '@' not in user_data['email']:
+     if '@' not in email or '.' not in email.split('@')[1]:
          raise ValueError("Invalid email")
      
-     # Check username
-     if not user_data.get('username'):
+     # 验证用户名
+     username = user_data.get('username', '').strip()
+     if not username:
          raise ValueError("Username required")
+     if len(username) < 3:
+         raise ValueError("Username too short")
+     if not username.isalnum():
+         raise ValueError("Username must be alphanumeric")
      
      return True
```

**问题：**
- 在 bug 修复范围外“改进”了邮箱验证
- 添加了无人要求的用户名验证
- 修改了注释
- 添加了 docstring

**✅ 更合适的处理方式（精准）**

```diff
  def validate_user(user_data):
      # Check email format
-     if not user_data.get('email'):
+     email = user_data.get('email', '')
+     if not email or not email.strip():
          raise ValueError("Email required")
      
      # Basic email validation
-     if '@' not in user_data['email']:
+     if '@' not in email:
          raise ValueError("Invalid email")
      
      # Check username
      if not user_data.get('username'):
          raise ValueError("Username required")
      
      return True
```

**只改动：** 修复空邮箱处理所必需的几行。

### 示例 2：风格漂移

**用户请求：** “给上传函数添加日志”

**❌ LLM 常见做法（重新格式化所有内容）**

```diff
- def upload_file(file_path, destination):
+ def upload_file(file_path: str, destination: str) -> bool:
+     """上传文件到目标地址并记录日志。"""
+     logger.info(f"Uploading {file_path} to {destination}")
+     
      try:
-         with open(file_path, 'rb') as f:
-             data = f.read()
-         
-         response = requests.post(destination, files={'file': data})
-         
-         if response.status_code == 200:
-             return True
-         else:
-             return False
+         with open(file_path, "rb") as f:
+             data = f.read()
+         response = requests.post(destination, files={"file": data})
+         success = response.status_code == 200
+         
+         if success:
+             logger.info(f"Upload successful: {file_path}")
+         else:
+             logger.error(f"Upload failed: {file_path}, status={response.status_code}")
+         
+         return success
      except Exception as e:
-         print(f"Error: {e}")
+         logger.exception(f"Upload error for {file_path}: {e}")
          return False
```

**问题：**
- 把单引号改成双引号
- 添加了无人要求的类型标注
- 添加了 docstring
- 重新格式化空行
- 改写了布尔返回逻辑

**✅ 更合适的处理方式（匹配现有风格）**

```diff
+ import logging
+ 
+ logger = logging.getLogger(__name__)
+ 
  def upload_file(file_path, destination):
+     logger.info(f'Starting upload: {file_path}')
      try:
          with open(file_path, 'rb') as f:
              data = f.read()
          
          response = requests.post(destination, files={'file': data})
          
          if response.status_code == 200:
+             logger.info(f'Upload successful: {file_path}')
              return True
          else:
+             logger.error(f'Upload failed: {file_path}, status={response.status_code}')
              return False
      except Exception as e:
-         print(f"Error: {e}")
+         logger.exception(f'Upload error: {file_path}')
          return False
```

**匹配了：** 单引号、无类型标注、现有布尔模式、现有空行风格。

---

## 4. 目标驱动执行

### 示例 1：模糊目标与可验证目标

**用户请求：** “修复认证系统”

**❌ LLM 常见做法（模糊方案）**

```
我会这样修复认证系统：
1. 阅读代码
2. 找出问题
3. 做一些改进
4. 测试改动

[在没有明确成功标准的情况下开始修改]
```

**✅ 更合适的处理方式（可验证目标）**

```
要“修复认证系统”，我需要先定义成功标准。
我们要解决的具体问题是什么？

如果问题是“用户修改密码后仍保持登录”：

计划：
1. 编写测试：修改密码 → 验证旧会话失效
   验证：测试失败（复现 bug）

2. 实现：修改密码时让会话失效
   验证：测试通过

3. 检查边界情况：多个活跃会话、并发修改
   验证：补充测试通过

4. 验证无回归：现有认证测试仍通过
   验证：完整测试套件通过

当前认证测试覆盖率：[展示覆盖率]
你看到的具体认证问题是什么？
```

### 示例 2：带验证的多步骤任务

**用户请求：** “给 API 添加限流”

**❌ LLM 常见做法（一次做完）**

```python
# 一次实现完整限流，包括 Redis、多种策略、
# 配置系统和监控，单个提交 300 行
# 没有清晰的验证步骤
```

**✅ 更合适的处理方式（增量推进并验证）**

```
限流计划：

1. 添加基础内存限流（单个端点）
   验证：
   - 测试：100 个请求 → 前 10 个成功，其余返回 429
   - 手动：curl 同一端点 11 次，看到限流错误

2. 抽取为中间件（应用到所有端点）
   验证：
   - 测试：限流同时作用于 /users 和 /posts
   - 现有端点测试仍通过

3. 添加 Redis 后端（支持多服务实例）
   验证：
   - 测试：应用重启后限流状态仍可用
   - 测试：两个应用实例共享限流计数器

4. 添加配置（每个端点不同限额）
   验证：
   - 测试：/search 允许 10/min，/users 允许 100/min
   - 配置文件能正确解析

每一步都可以独立验证和部署。
是否从第 1 步开始？
```

### 示例 3：先写测试再验证

**用户请求：** “分数重复时排序会坏掉”

**❌ LLM 常见做法（未复现就修）**

```python
# 立即修改排序逻辑，没有先确认 bug
def sort_scores(scores):
    return sorted(scores, key=lambda x: (-x['score'], x['name']))
```

**✅ 更合适的处理方式（先复现）**

```python
# 1. 先写一个能复现问题的测试
def test_sort_with_duplicate_scores():
    """测试多个项目分数相同时的排序。"""
    scores = [
        {'name': 'Alice', 'score': 100},
        {'name': 'Bob', 'score': 100},
        {'name': 'Charlie', 'score': 90},
    ]
    
    result = sort_scores(scores)
    
    # bug：重复分数的顺序不稳定
    # 多次运行该测试时，结果应保持一致
    assert result[0]['score'] == 100
    assert result[1]['score'] == 100
    assert result[2]['score'] == 90

# 验证：运行测试 10 次 → 因顺序不稳定而失败

# 2. 再用稳定排序修复
def sort_scores(scores):
    """按分数降序排序，分数相同时按姓名升序排序。"""
    return sorted(scores, key=lambda x: (-x['score'], x['name']))

# 验证：测试稳定通过
```

---

## 反模式总结

| 原则 | 反模式 | 修复 |
|-----------|-------------|-----|
| 编码前思考 | 默默假设文件格式、字段、范围 | 明确列出假设，询问澄清 |
| 简洁优先 | 为单一折扣计算引入策略模式 | 先用一个函数，等复杂度真实出现再调整 |
| 精准修改 | 修复 bug 时顺手重排引号、添加类型标注 | 只修改能解决已报告问题的代码行 |
| 目标驱动 | “我会检查并改进代码” | “为 bug X 写测试 → 让测试通过 → 验证无回归” |

## 核心洞察

“过度复杂”的示例表面上并不明显错误，它们使用了设计模式和最佳实践。问题在于**时机**：在真正需要前就引入复杂度，会导致：

- 代码更难理解
- 引入更多 bug
- 实现时间更长
- 测试更困难

“简单”的版本具有这些优势：
- 更容易理解
- 更快实现
- 更容易测试
- 后续复杂度真实出现时可以再重构
