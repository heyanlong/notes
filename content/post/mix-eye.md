---
title: "使用tensorflow和OpenCV合并人眼"
date: 2019-10-30T18:33:22+08:00
draft: false
tags: ["MacOS", "OpenCV", "Python", "tensorflow"]
keywords: ["MacOS", "OSX", "python", "ai换脸", "合影修复"]

categories:
  - "计算机视觉"
  - "AI"
---

在很多时候对摄影师而言，合影一直是摄影师之痛。很多时候的场景是，都站好了，拍完一看有人闭眼了，这个时候是再拍一张还是就这样呢？？

如题，我们要合并人眼，听起来很神奇的一个事情，目的是在合影时进行连拍，然后利用tensorflow训练的模型替换掉闭眼的哥们，最终达到所有人睁眼的效果。

目标是开发一个计算机视觉程序，检测眼睛是打开还是关闭的状态。

源码已上传到[GitHub仓库](https://github.com/heyanlong/mix-face)

搞起～

## 第一阶段

首选需要训练一个cnn，这里需要使用tensorflow和keras库，先安装他们

```shell
pip install tensorflow
pip install keras
```

开始准备一些睁眼闭眼的图片，并将图片裁剪为24 x 24 大小，准好好数据之后，开始训练一个具备睁眼闭眼检测的二元分类器。

睁眼闭眼图片可以在我仓库的dataset文件夹中找到。

编辑train.py 引入所需的库
```python
from keras.models import Sequential
from keras.layers import Conv2D
from keras.layers import AveragePooling2D
from keras.layers import Flatten
from keras.layers import Dense
from keras.preprocessing.image import ImageDataGenerator
```

然后加载数据集

```python
def collect():
	train_datagen = ImageDataGenerator(
			rescale=1./255,
			shear_range=0.2,
			horizontal_flip=True, 
		)

	val_datagen = ImageDataGenerator(
			rescale=1./255,
			shear_range=0.2,
			horizontal_flip=True,		)

	train_generator = train_datagen.flow_from_directory(
	    directory="dataset/train",
	    target_size=(IMG_SIZE, IMG_SIZE),
	    color_mode="grayscale",
	    batch_size=32,
	    class_mode="binary",
	    shuffle=True,
	    seed=42
	)

	val_generator = val_datagen.flow_from_directory(
	    directory="dataset/val",
	    target_size=(IMG_SIZE, IMG_SIZE),
	    color_mode="grayscale",
	    batch_size=32,
	    class_mode="binary",
	    shuffle=True,
	    seed=42
	)
	return train_generator, val_generator
```

制作模型

```python
def train(train_generator, val_generator):
	STEP_SIZE_TRAIN=train_generator.n//train_generator.batch_size
	STEP_SIZE_VALID=val_generator.n//val_generator.batch_size

	print('[LOG] Intialize Neural Network')
	
	model = Sequential()

	model.add(Conv2D(filters=6, kernel_size=(3, 3), activation='relu', input_shape=(IMG_SIZE,IMG_SIZE,1)))
	model.add(AveragePooling2D())

	model.add(Conv2D(filters=16, kernel_size=(3, 3), activation='relu'))
	model.add(AveragePooling2D())

	model.add(Flatten())

	model.add(Dense(units=120, activation='relu'))

	model.add(Dense(units=84, activation='relu'))

	model.add(Dense(units=1, activation = 'sigmoid'))


	model.compile(loss='binary_crossentropy', optimizer='adam', metrics=['accuracy'])

	model.fit_generator(generator=train_generator,
	                    steps_per_epoch=STEP_SIZE_TRAIN,
	                    validation_data=val_generator,
	                    validation_steps=STEP_SIZE_VALID,
	                    epochs=20
	)
	save_model(model)
```

ok，现在已经拥有了简单的cnn，并且具备睁眼闭眼识别能力。

## 第二阶段

程序处理思路


1. 识别出第一张照片所有人脸位置
1. 对同一个位置的人脸，对每一张图片相同位置进行扣图，得到一个人多张人脸
1. 对每一个人的多张人脸进行眼睛打开关闭评分
1. 得分最高的人脸替换掉第一张图片同样位置的人脸

传入多张照片，并在第一张照片中识别所有人脸，然后根据第一张人脸坐标扣出同一个人的其他人脸
```python
def upload():
    images = request.files.getlist('img[]')  # a upload files
    print("******", images)

    if len(images):
        im = []
        # save
        for i in range(len(images)):
            image = images[i]
            input = config.upload_dir + image.filename

            im.append({
                'raw': image,
                'path': input,
            })

            try:
                image.save(input)
            except Exception as e:
                print("Error: ", e)

        # read first image as base image
        first_image_path = im[0]['path']
        first_image = cv2.imread(first_image_path)

        gray = cv2.cvtColor(first_image, cv2.COLOR_RGB2GRAY)

        faces = face_cascade.detectMultiScale(gray)

        # save first image faces rect
        im[0]['faces'] = faces

        mini_face = []
        for i in range(len(faces)):
            mini_face.append({
                'ok': {},
                'face_list': []
            })

        for j in range(len(im)):
            img = cv2.imread(im[j]['path'])
            for i in range(len(faces)):
                face = faces[i]
                rect = np.rint([face[0], face[1], face[0] + face[2], face[1] + face[3]])
                rect = rect.astype(int)
                f = img[rect[1]:rect[3], rect[0]:rect[2]]
                mini_face[i]['face_list'].append({
                    'image': f,
                    'rect': face,
                    'np_rect': rect
                })

        for i in range(len(mini_face)):
            faces = mini_face[i]['face_list']
            for j in range(len(faces)):
                face = faces[j]
                max_face_image = tool.img_resize_to_target_white(face['image'])
                left_eye = right_eye = 0
                prediction = 0

                # detect eyes
                eyes = cropEyes(max_face_image)
                if eyes is None:
                    print("eyes is null")
                else:
                    left_eye, right_eye = eyes
                    prediction = (model.predict(cnnPreprocess(left_eye)) + model.predict(
                        cnnPreprocess(right_eye))) / 2.0

                mini_face[i]['face_list'][j]['prediction'] = prediction

                if 'prediction' not in mini_face[i]['ok']:
                    mini_face[i]['ok']['face'] = face
                    mini_face[i]['ok']['prediction'] = prediction
                else:
                    if mini_face[i]['ok']['prediction'] < prediction:
                        mini_face[i]['ok']['face'] = face
                        mini_face[i]['ok']['prediction'] = prediction

        # merge
        for item in mini_face:
            ok = item['ok']
            print(ok['prediction'])
            first_image = merge_image(first_image, ok['face']['image'], ok['face']['rect'])

        cv2.putText(first_image, 'mix@magvii ' + time.strftime('%Y-%m-%d %H:%M:%S', time.localtime(time.time())), (150, 150), cv2.FONT_HERSHEY_SIMPLEX, 5, (0, 0, 0), 5, cv2.LINE_AA)
        cv2.imwrite("mix/dist/img/merged_image.jpg", first_image)
        return '{"code": 1000}'
    else:
        return '{"code": 9999}'
```

由于剪裁出的人脸过小，进行人脸组评分的时候识别器识别失败，我们进行人脸放大，得到比当前人脸大两倍的图像
```python
def img_resize_to_target_white(input_image):
    img = input_image
    h = img.shape[0]
    w = img.shape[1]
    target = np.ones((2 * h, 2 * w), dtype=np.uint8) * 255

    half_h = int(h / 2)
    half_w = int(w / 2)
    ret = cv2.cvtColor(target, cv2.COLOR_GRAY2BGR)
    for i in range(2 * h):
        for j in range(2 * w):
            if (half_h < i) and (i < h + half_h) and (half_w < j) and (j < w + half_w):
                ret[i, j, 0] = img[i - half_h, j - half_w, 0]
                ret[i, j, 1] = img[i - half_h, j - half_w, 1]
                ret[i, j, 2] = img[i - half_h, j - half_w, 2]
            else:
                ret[i, j, 0] = 255
                ret[i, j, 1] = 255
                ret[i, j, 2] = 255

    return ret
```

根据放大的图像找到左眼和右眼，并处理成24 x 24的图片
```python
def cropEyes(frame):
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    # detect the face at grayscale image
    te = detect(gray)

    if len(te) == 0:
        return None
    elif len(te) > 1:
        face = te[0]
    elif len(te) == 1:
        [face] = te

    # keep the face region from the whole frame
    face_rect = dlib.rectangle(left=int(face[0]), top=int(face[1]),
                               right=int(face[2]), bottom=int(face[3]))
    # determine the facial landmarks for the face region
    shape = predictor(gray, face_rect)
    shape = face_utils.shape_to_np(shape)

    (rStart, rEnd) = face_utils.FACIAL_LANDMARKS_IDXS["left_eye"]
    (lStart, lEnd) = face_utils.FACIAL_LANDMARKS_IDXS["right_eye"]

    # extract the left and right eye coordinates
    leftEye = shape[lStart:lEnd]
    rightEye = shape[rStart:rEnd]

    l_uppery = min(leftEye[1:3, 1])
    l_lowy = max(leftEye[4:, 1])
    minxl = leftEye[0][0]
    maxxl = leftEye[3][0]
    higl = (maxxl - minxl) / (24 / 24)
    minyl = l_uppery + (((l_lowy - l_uppery) - higl) / 2)
    maxyl = minyl + higl

    # crop the eye rectangle from the frame
    left_eye_rect = np.rint([minxl, minyl, maxxl, maxyl])
    left_eye_rect = left_eye_rect.astype(int)
    left_eye_image = gray[(left_eye_rect[1]):left_eye_rect[3], (left_eye_rect[0]):left_eye_rect[2]]

    # same as left eye at right eye
    r_uppery = min(rightEye[1:3, 1])
    r_lowy = max(rightEye[4:, 1])
    minxr = rightEye[0][0]
    maxxr = rightEye[3][0]
    higr = (maxxr - minxr) / (24 / 24)
    minyr = r_uppery + (((r_lowy - r_uppery) - higr) / 2)
    maxyr = minyr + higr

    right_eye_rect = np.rint([minxr, minyr, maxxr, maxyr])
    right_eye_rect = right_eye_rect.astype(int)
    right_eye_image = gray[right_eye_rect[1]:right_eye_rect[3], right_eye_rect[0]:right_eye_rect[2]]

    # if it doesn't detect left or right eye return None
    if 0 in left_eye_image.shape or 0 in right_eye_image.shape:
        return None
    # resize for the conv net
    left_eye_image = cv2.resize(left_eye_image, (24, 24))
    right_eye_image = cv2.resize(right_eye_image, (24, 24))
    right_eye_image = cv2.flip(right_eye_image, 1)
    # return left and right eye
    return left_eye_image, right_eye_image
```

进行人眼评分，分值在0到1直接，0代表绝对闭眼，1代表绝对睁眼
```python
prediction = (model.predict(cnnPreprocess(left_eye)) + model.predict(
                        cnnPreprocess(right_eye))) / 2.0
```

最佳人脸与第一张人脸合并

```python
def merge_image(lack_face_img, face_img, rect):
    (x, y, w, h) = rect
    #x += 10
    #y += 10
    height, width, _ = face_img.shape
    for i in range(height):
        for j in range(width):
            lack_face_img[i + y, j + x, 0] = face_img[i, j, 0]
            lack_face_img[i + y, j + x, 1] = face_img[i, j, 1]
            lack_face_img[i + y, j + x, 2] = face_img[i, j, 2]
    return lack_face_img
```

到此为止，我们就把多张质量不一的照片合并成一张质量最好的照片了～
