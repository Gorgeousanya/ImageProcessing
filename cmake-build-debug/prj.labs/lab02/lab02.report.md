## Работа 2. Исследование каналов и JPEG-сжатия
автор: Полевой Д.В.
дата: 2022-02-16T18:03:40

<!-- url: https://gitlab.com/2021-misis-spring/polevoy_d_v/-/tree/master/prj.labs/lab02 -->

### Задание
1. В качестве тестового использовать изображение data/cross_0256x0256.png
2. Сохранить тестовое изображение в формате JPEG с качеством 25%.
3. Используя cv::merge и cv::split сделать "мозаику" с визуализацией каналов для исходного тестового изображения и JPEG-версии тестового изображения
- левый верхний - трехканальное изображение
- левый нижний - монохромная (черно-зеленая) визуализация канала G
- правый верхний - монохромная (черно-красная) визуализация канала R
- правый нижний - монохромная (черно-синяя) визуализация канала B
4. Результы сохранить для вставки в отчет
5. Сделать мозаику из визуализации гистограммы для исходного тестового изображения и JPEG-версии тестового изображения, сохранить для вставки в отчет.

### Результаты

![](cross_0256x0256_025.jpg)
Рис. 1. Тестовое изображение после сохранения в формате JPEG с качеством 25%

![](cross_0256x0256_png_channels.png)
Рис. 2. Визуализация каналов исходного тестового изображения

![](cross_0256x0256_jpg_channels.png)
Рис. 3. Визуализация каналов JPEG-версии тестового изображения

![](cross_0256x0256_hists.png)
Рис. 3. Визуализация гистограм исходного и JPEG-версии тестового изображения

### Текст программы

```cpp
#include <opencv2/opencv.hpp>

int main() {
    cv::Mat img = cv::imread( "/Users/annaplokhotnyuk/Downloads/plokhotnyuk_a_d/data/cross_0256x0256.png");
    cv::imwrite("test.jpeg", img, {cv::IMWRITE_JPEG_QUALITY, 25});
    std::vector<cv::Mat> rgbChannels(3);
    split(img, rgbChannels);

    // Show individual channels
    cv::Mat g, red_img;
    g = cv::Mat::zeros(cv::Size(img.cols, img.rows), CV_8UC1);

    std::vector<cv::Mat> channels_r;
    channels_r.push_back(g);
    channels_r.push_back(g);
    channels_r.push_back(rgbChannels[2]);

    cv::merge(channels_r,  red_img);

    cv::imwrite("red.jpeg", red_img);

    cv::Mat green_img;
    std::vector<cv::Mat> channels_g;
    channels_g.push_back(g);
    channels_g.push_back(rgbChannels[1]);
    channels_g.push_back(g);

    cv::merge(channels_g,  green_img);

    cv::imwrite("green.jpeg", green_img);

    cv::Mat blue_img;
    std::vector<cv::Mat> channels_b;
    channels_b.push_back(rgbChannels[0]);
    channels_b.push_back(g);
    channels_b.push_back(g);

    cv::merge(channels_b,  blue_img);

    cv::imwrite("blue.jpeg", blue_img);


}
```
