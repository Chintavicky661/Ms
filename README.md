multilayer MNIST:


import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.linear_model import Perceptron
from sklearn.datasets import fetch_openml
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import accuracy_score
from mlxtend.plotting import plot_decision_regions
from sklearn.decomposition import PCA

mnist = fetch_openml('mnist_784', version=1)
X, y = mnist.data, mnist.target
y = (y == '0').astype(int)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

scaler = StandardScaler()
X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)

p = Perceptron(max_iter=1000, tol=1e-3, random_state=42)
p.fit(X_train, y_train)

y_pred = p.predict(X_test)
accuracy = accuracy_score(y_test, y_pred)
print("Accuracy score:", accuracy)

pca = PCA(n_components=2)
X_train_pca = pca.fit_transform(X_train)
p.fit(X_train_pca, y_train.to_numpy())

plt.figure(figsize=(8, 6))
plot_decision_regions(X_train_pca, y_train.to_numpy(), clf=p, legend=2)
plt.title("Perceptron Decision Boundary (Reduced Features)")
plt.xlabel("Principal Component 1")
plt.ylabel("Principal Component 2")
plt.show()


multiclass:

import tensorflow as tf
from tensorflow.keras import layers
import matplotlib.pyplot as plt

reuters = tf.keras.datasets.reuters
(train_data, train_labels), (test_data, test_labels) = reuters.load_data(num_words=10000)

word_index = reuters.get_word_index()

train_data = tf.keras.preprocessing.sequence.pad_sequences(train_data, maxlen=100)
test_data = tf.keras.preprocessing.sequence.pad_sequences(test_data, maxlen=100)

model = tf.keras.Sequential([
    layers.Embedding(10000, 46),
    layers.LSTM(64, return_sequences=True),
    layers.GlobalAveragePooling1D(),
    layers.Dense(64, activation='relu'),
    layers.Dense(46, activation='softmax')
])

model.compile(
    optimizer='adam',
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

history = model.fit(
    train_data,
    train_labels,
    epochs=10,
    validation_data=(test_data, test_labels)
)

test_loss, test_acc = model.evaluate(test_data, test_labels)
print('Test accuracy:', test_acc)

plt.figure(figsize=(12, 5))

plt.subplot(1, 2, 1)
plt.plot(history.history['accuracy'], label='Train Accuracy')
plt.plot(history.history['val_accuracy'], label='Validation Accuracy')
plt.xlabel('Epochs')
plt.ylabel('Accuracy')
plt.title('Training and Validation Accuracy')
plt.legend()

plt.subplot(1, 2, 2)
plt.plot(history.history['loss'], label='Train Loss')
plt.plot(history.history['val_loss'], label='Validation Loss')
plt.xlabel('Epochs')
plt.ylabel('Loss')
plt.title('Training and Validation Loss')
plt.legend()

plt.show()

onehot:

import pandas as pd
from sklearn.preprocessing import OneHotEncoder

data = {
    'Employee id': [10, 20, 15, 25, 30],
    'Gender': ['M', 'F', 'F', 'M', 'F'],
    'Remarks': ['Good', 'Nice', 'Good', 'Great', 'Nice']
}
df = pd.DataFrame(data)
print(f"Original Employee DataFrame : \n{df}\n")

df_pandas_encoded = pd.get_dummies(df, columns=['Gender', 'Remarks'], drop_first=True)
print(f"One-Hot Encoded DataFrame using pd.get_dummies() :\n{df_pandas_encoded}\n")

encoder = OneHotEncoder(sparse_output=False)
categorical_colums = ['Gender', 'Remarks']
one_hot_encoded = encoder.fit_transform(df[categorical_colums])
one_hot_df = pd.DataFrame(one_hot_encoded,
                          columns=encoder.get_feature_names_out(categorical_colums))
df_sklearn_encoded = pd.concat([df.drop(categorical_colums, axis=1), one_hot_df], axis=1)
print(f"One-Hot Encoded data using Scikit-Learn:\n{df_sklearn_encoded}\n")


CNN;

import tensorflow as tf
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Conv2D, MaxPooling2D, Flatten, Dense, Dropout

mnist = tf.keras.datasets.mnist
(x_train, y_train), (x_test, y_test) = mnist.load_data()

x_train, x_test = x_train / 255.0, x_test / 255.0
x_train = x_train.reshape(-1, 28, 28, 1)
x_test = x_test.reshape(-1, 28, 28, 1)

model = Sequential([
    Conv2D(32, (3, 3), activation='relu', input_shape=(28, 28, 1)),
    MaxPooling2D((2, 2)),
    Conv2D(64, (3, 3), activation='relu'),
    MaxPooling2D((2, 2)),
    Flatten(),
    Dense(128, activation='relu'),
    Dropout(0.2),
    Dense(10, activation='softmax')])

model.compile(
    optimizer='adam',
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

model.fit(x_train, y_train, epochs=10)
model.evaluate(x_test, y_test)




VGG:

import numpy as np
from keras.preprocessing import image
from keras.applications.vgg16 import VGG16, preprocess_input, decode_predictions

model = VGG16(weights='imagenet')

img_path = 'io.jpg'
img = image.load_img(img_path, target_size=(224, 224))
img_array = image.img_to_array(img)
img_array = np.expand_dims(img_array, axis=0)
img_array = preprocess_input(img_array)

predictions = model.predict(img_array)
decoded_predictions = decode_predictions(predictions, top=3)[0]

print("Top predictions:")
for i, (imagenet_id, label, score) in enumerate(decoded_predictions):
    print(f"{i + 1}: {label} ({score:.2f})")
