# tasks/models.py
from django.db import models
from django.contrib.auth.models import User

class Task(models.Model):
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    due_date = models.DateTimeField(null=True, blank=True)
    completed = models.BooleanField(default=False)
    owner = models.ForeignKey(User, on_delete=models.CASCADE)

    def __str__(self):
        return self.title
# tasks/views.py
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth.decorators import login_required
from .models import Task
from .forms import TaskForm

@login_required
def task_list(request):
    tasks = Task.objects.filter(owner=request.user)
    return render(request, 'tasks/task_list.html', {'tasks': tasks})

@login_required
def task_create(request):
    if request.method == "POST":
        form = TaskForm(request.POST)
        if form.is_valid():
            task = form.save(commit=False)
            task.owner = request.user
            task.save()
            return redirect('task_list')
    else:
        form = TaskForm()
    return render(request, 'tasks/task_form.html', {'form': form})

@login_required
def task_update(request, pk):
    task = get_object_or_404(Task, pk=pk, owner=request.user)
    if request.method == "POST":
        form = TaskForm(request.POST, instance=task)
        if form.is_valid():
            form.save()
            return redirect('task_list')
    else:
        form = TaskForm(instance=task)
    return render(request, 'tasks/task_form.html', {'form': form})

@login_required
def task_delete(request, pk):
    task = get_object_or_404(Task, pk=pk, owner=request.user)
    if request.method == "POST":
        task.delete()
        return redirect('task_list')
    return render(request, 'tasks/task_confirm_delete.html', {'task': task})
# tasks/views.py
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth.decorators import login_required
from .models import Task
from .forms import TaskForm

@login_required
def task_list(request):
    tasks = Task.objects.filter(owner=request.user)
    return render(request, 'tasks/task_list.html', {'tasks': tasks})

@login_required
def task_create(request):
    if request.method == "POST":
        form = TaskForm(request.POST)
        if form.is_valid():
            task = form.save(commit=False)
            task.owner = request.user
            task.save()
            return redirect('task_list')
    else:
        form = TaskForm()
    return render(request, 'tasks/task_form.html', {'form': form})

@login_required
def task_update(request, pk):
    task = get_object_or_404(Task, pk=pk, owner=request.user)
    if request.method == "POST":
        form = TaskForm(request.POST, instance=task)
        if form.is_valid():
            form.save()
            return redirect('task_list')
    else:
        form = TaskForm(instance=task)
    return render(request, 'tasks/task_form.html', {'form': form})

@login_required
def task_delete(request, pk):
    task = get_object_or_404(Task, pk=pk, owner=request.user)
    if request.method == "POST":
        task.delete()
        return redirect('task_list')
    return render(request, 'tasks/task_confirm_delete.html', {'task': task})
# tasks/forms.py
from django import forms
from .models import Task

class TaskForm(forms.ModelForm):
    class Meta:
        model = Task
        fields = ['title', 'description', 'due_date', 'completed']
# tasks/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.task_list, name='task_list'),
    path('create/', views.task_create, name='task_create'),
    path('update/<int:pk>/', views.task_update, name='task_update'),
    path('delete/<int:pk>/', views.task_delete, name='task_delete'),
]
<!-- tasks/templates/tasks/task_list.html -->
<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8">
  <title>قائمة المهام</title>
</head>
<body>
  <h1>مهامي</h1>
  <a href="{% url 'task_create' %}">إنشاء مهمة جديدة</a>
  <ul>
    {% for task in tasks %}
      <li>
        <strong>{{ task.title }}</strong> - {{ task.created_at|date:"d/m/Y" }}
        {% if task.completed %}
          (مكتملة)
        {% endif %}
        <a href="{% url 'task_update' task.pk %}">تعديل</a>
        <a href="{% url 'task_delete' task.pk %}">حذف</a>
      </li>
    {% empty %}
      <li>لا توجد مهام حتى الآن.</li>
    {% endfor %}
  </ul>
</body>
</html>
// models/Product.js
const mongoose = require('mongoose');

const ProductSchema = new mongoose.Schema({
  name: { type: String, required: true },
  description: String,
  price: { type: Number, required: true },
  inStock: { type: Boolean, default: true },
  createdAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('Product', ProductSchema);
// routes/productRoutes.js
const express = require('express');
const router = express.Router();
const Product = require('../models/Product');

// جلب جميع المنتجات
router.get('/', async (req, res) => {
  try {
    const products = await Product.find();
    res.json(products);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
});

// إنشاء منتج جديد
router.post('/', async (req, res) => {
  const product = new Product({
    name: req.body.name,
    description: req.body.description,
    price: req.body.price,
    inStock: req.body.inStock
  });
  try {
    const newProduct = await product.save();
    res.status(201).json(newProduct);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
});

// تحديث منتج
router.patch('/:id', async (req, res) => {
  try {
    const product = await Product.findById(req.params.id);
    if (req.body.name != null) {
      product.name = req.body.name;
    }
    if (req.body.description != null) {
      product.description = req.body.description;
    }
    if (req.body.price != null) {
      product.price = req.body.price;
    }
    if (req.body.inStock != null) {
      product.inStock = req.body.inStock;
    }
    const updatedProduct = await product.save();
    res.json(updatedProduct);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
});

// حذف منتج
router.delete('/:id', async (req, res) => {
  try {
    await Product.findByIdAndDelete(req.params.id);
    res.json({ message: 'تم حذف المنتج' });
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
});

module.exports = router;
// server.js
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');
const productRoutes = require('./routes/productRoutes');

const app = express();
const PORT = process.env.PORT || 3000;

// إعداد Body Parser لتحليل الطلبات
app.use(bodyParser.json());

// ربط المسارات
app.use('/products', productRoutes);

// الاتصال بقاعدة البيانات MongoDB
mongoose.connect('mongodb://localhost/ecommerce', { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => {
    console.log('تم الاتصال بقاعدة البيانات');
    app.listen(PORT, () => console.log(`يعمل التطبيق على المنفذ ${PORT}`));
  })
  .catch(err => console.error(err));
// server.js
const express = require('express');
const app = express();
const http = require('http').createServer(app);
const io = require('socket.io')(http);
const path = require('path');

// تقديم ملفات الواجهة الثابتة
app.use(express.static(path.join(__dirname, 'public')));

// عند الاتصال، التعامل مع الأحداث
io.on('connection', (socket) => {
  console.log('مستخدم جديد متصل');

  socket.on('chat message', (msg) => {
    // إعادة إرسال الرسالة لجميع المستخدمين
    io.emit('chat message', msg);
  });

  socket.on('disconnect', () => {
    console.log('مستخدم قد فصل الاتصال');
  });
});

http.listen(3000, () => {
  console.log('يعمل التطبيق على المنفذ 3000');
});
<!-- public/index.html -->
<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8">
  <title>تطبيق الدردشة الفورية</title>
  <style>
    ul { list-style-type: none; padding: 0; }
    li { margin-bottom: 10px; }
  </style>
</head>
<body>
  <ul id="messages"></ul>
  <form id="chat-form">
    <input id="m" autocomplete="off" placeholder="أدخل رسالتك" /><button>إرسال</button>
  </form>
  
  <script src="/socket.io/socket.io.js"></script>
  <script>
    const socket = io();
    const form = document.getElementById('chat-form');
    const input = document.getElementById('m');
    const messages = document.getElementById('messages');

    form.addEventListener('submit', (e) => {
      e.preventDefault();
      if (input.value) {
        socket.emit('chat message', input.value);
        input.value = '';
      }
    });

    socket.on('chat message', (msg) => {
      const item = document.createElement('li');
      item.textContent = msg;
      messages.appendChild(item);
      window.scrollTo(0, document.body.scrollHeight);
    });
  </script>
</body>
</html>
// App.js
import React, { useState } from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';

export default function App() {
  const [steps, setSteps] = useState(0);

  const increaseSteps = () => {
    // هنا نقوم بزيادة عدد الخطوات بمقدار ثابت للتوضيح
    setSteps(prevSteps => prevSteps + 1000);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.text}>عدد الخطوات: {steps}</Text>
      <Button title="زيادة الخطوات" onPress={increaseSteps} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#f5fcff'
  },
  text: {
    fontSize: 24,
    marginBottom: 20
  }
});
