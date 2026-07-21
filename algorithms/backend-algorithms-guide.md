# 🚀 راهنمای جامع الگوریتم‌ها و ساختارهای داده در بک‌اند و مصاحبه‌های تخصصی

این سند خلاصه‌ای از مفاهیم، سناریوهای معماری، ساختارهای داده و الگوریتم‌های پرکاربرد در مهندسی بک‌اند، System Design و مصاحبه‌های فنی است.

---

## 📑 فهرست مطالب
1. [گراف‌ها (Graphs) و معماری میکروسرویس](#1-گراف‌ها-graphs-و-معماری-میکروسرویس)
2. [درخت‌ها (Trees) و ایندکس‌گذاری دیتابیس (B-Tree)](#2-درخت‌ها-trees-و-ایندکس‌گذاری-دیتابیس-b-tree)
3. [الگوریتم دایکسترا (Dijkstra) و مسیریابی](#3-الگوریتم-دایکسترا-dijkstra-و-مسیریابی)
4. [هشینگ یکنواخت (Consistent Hashing)](#4-هشینگ-یکنواخت-consistent-hashing)
5. [محدودسازی نرخ درخواست (Token Bucket Rate Limiter)](#5-محدودسازی-نرخ-درخواست-token-bucket-rate-limiter)

---

## ۱. گراف‌ها (Graphs) و معماری میکروسرویس

### 💡 مفهوم در دنیای واقعی
ارتباطات بین میکروسرویس‌ها، شبکه و پایپ‌لاین‌های CI/CD بهترین نمونه برای مدل‌سازی به صورت **گراف** هستند:
* **گره‌ها (Nodes / Vertices):** سرویس‌ها یا کامپوننت‌های سیستم.
* **یال‌ها (Edges):** ارتباطات، درخواست‌های HTTP/gRPC یا وابستگی‌ها.

### ⚠️ مسئله Cycle (دور) و وابستگی چرخشی
وجود یک حلقه بین سرویس‌ها (مثلاً $A \rightarrow B \rightarrow C \rightarrow A$) باعث Infinite Loop و Crash سیستم می‌شود.
* **راهکار کشف دور:** استفاده از **پیمایش عمقی (DFS - Depth First Search)**.
* **ترتیب اجرای سرویس‌ها:** استفاده از **Topological Sort** (بر پایه DFS) در ابزارهایی مثل Docker Compose یا Kubernetes برای پیدا کردن ترتیب صحیح روشن شدن کانتینرها.

### 🧠 ذخیره‌سازی در حافظه: Adjacency List vs Adjacency Matrix
برای ۱۰۰۰ میکروسرویس خلوت (Sparse Graph):
* **Adjacency Matrix ($O(V^2)$):** هدر رفت شدید حافظه به دلیل خانه‌های خالی.
* **Adjacency List ($O(V + E)$) [انتخاب برتر]:** استفاده از یک `Map` که مقادیر آن یک لیست یا **Priority Queue** (برای تعیین اولویت فراخوانی سرویس‌های حیاتی) است.

---

## ۲. درخت‌ها (Trees) و ایندکس‌گذاری دیتابیس (B-Tree)

### 📚 ایندکس چیست؟
ایندکس مثل دفترچه فهرست انتهای کتاب است که آدرس دقیق هر داده روی دیسک را نگه می‌دارد تا از **Full Table Scan ($O(n)$)** جلوگیری کند.

### ⚖️ مقایسه Hash Index و B-Tree Index

| ویژگی | Hash Index (Map) | B-Tree Index |
| :--- | :--- | :--- |
| **جستجوی دقیق (`WHERE id = 5`)** | ⚡ $O(1)$ | 🚀 $O(\log n)$ |
| **جستجوی بازه‌ای (`WHERE age > 20`)** | ❌ $O(n)$ (نامرتب است) | ✅ $O(\log n)$ (کاملاً مرتب) |
| **مرتب‌سازی (`ORDER BY`)** | ❌ نیازمند سورت جداگانه | ✅ داده‌ها از قبل مرتب هستند |

### 🎯 ایندکس ترکیبی (Composite Index) و قانون Leftmost Prefix
در ایندکس ترکیبی `(user_id, order_date)`:
1. داده‌ها ابتدا بر اساس `user_id` دسته‌بندی می‌شوند.
2. درون هر کاربر، داده‌ها بر اساس `order_date` مرتب می‌شوند.
* **نکته کلیدی:** این ایندکس برای کوئری بر اساس `user_id` به تنهایی کار می‌کند، اما برای کوئری بر اساس `order_date` به تنهایی **کاربرد ندارد**.

---

## ۳. الگوریتم دایکسترا (Dijkstra) و مسیریابی

### 🧭 کاربرد
پیدا کردن **کوتاه‌ترین مسیر** در گراف‌های وزن‌دار (مثل پیدا کردن کمترین Latency شبکه یا ارزان‌ترین مسیر پستی).

* **استراتژی:** طمعکارانه (Greedy) با استفاده از **Priority Queue (Min-Heap)**.
* **پیچیدگی زمانی:** $O((V + E) \log V)$
* **نقطه ضعف:** روی گراف‌های با **وزن منفی (Negative Weight)** شکست می‌خورد (در آن حالت از **Bellman-Ford** استفاده می‌شود).

### 💻 پیاده‌سازی مختصر (Python)
```python
import heapq

def dijkstra(graph, start):
    distances = {node: float('inf') for node in graph}
    distances[start] = 0
    pq = [(0, start)]
    
    while pq:
        curr_dist, curr_node = heapq.heappop(pq)
        if curr_dist > distances[curr_node]:
            continue
            
        for neighbor, weight in graph[curr_node].items():
            dist = curr_dist + weight
            if dist < distances[neighbor]:
                distances[neighbor] = dist
                heapq.heappush(pq, (dist, neighbor))
    return distances
