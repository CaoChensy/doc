# Keras 

## 模型

### 输出可视化模型结构

```python
keras.utils.plot_model(model, 'multi_input_and_output_model.png', show_shapes=True)
```
## 优化

### Early Stop 模型终止条件

```python
# 忍耐参数（patience parameter），检查在某些训练周期中模型是否不断改进，当模型训练不再使模型更好时停止训练。
early_stop = keras.callbacks.EarlyStopping(monitor='val_loss', patience=10)

measure = model.fit(
    self.X_train, self.y_train,
    epochs=100,
    validation_data=(self.X_test, self.y_test),
    callbacks=[early_stop]
)
```

## 保存