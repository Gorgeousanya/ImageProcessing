## Работа 3. Яркостные преобразования
автор: Плохотнюк А. Д.
дата: 2022-02-26T17:37:27

<!-- url: https://gitlab.com/2021-misis-spring/polevoy_d_v/-/tree/master/prj.labs/lab03 -->

### Задание
1. В качестве тестового использовать изображение data/cross_0256x0256.png
2. Сгенерировать нетривиальную новую функцию преобразования яркости (не стоит использовать слишком простые и слишком простые функции).
3. Сгенерировать визуализацию функцию преобразования яркости в виде изображения размером 512x512, черные точки а белом фоне.
4. Преобразовать пиксели grayscale версии тестового изображения при помощи LUT для сгенерированной функции преобразования.
4. Преобразовать пиксели каждого канала тестового изображения при помощи LUT для сгенерированной функции преобразования.
5. Результы сохранить для вставки в отчет.

### Результаты

![](lab03_rgb.png)
Рис. 1. Исходное тестовое изображение

![](lab03_gre.png)
Рис. 2. Тестовое изображение greyscale

![](lab03_gre_res.png)
Рис. 3. Результат применения функции преобразования яркости для greyscale

![](lab03_rgb_res.png)
Рис. 4. Результат применения функции преобразования яркости для каналов

![](lab03_viz_func.png)
Рис. 5. Визуализация функции яркостного преобразования

### Текст программы

```cpp
#include <opencv2/opencv.hpp>


uchar brightness_func(const uchar& x) {
    return 25 * log(1 + x);
}

cv::Mat create_LUT(const cv::Mat& image) {
    cv::Mat look_up_table(cv::Mat::zeros(cv::Size{ 1, 256 }, CV_8UC1));
    for (int i = 0; i < 256; i++) {
        look_up_table.at<uchar>(i) = brightness_func(i);
    }
    return look_up_table;
}

cv::Mat func_graph(const cv::Mat& look_up_table) {
    cv::Mat graph(cv::Mat::ones(512, 512, CV_8UC1) * 255);
    double kw(512. / 256.);
    int maxVal = 1;
    for (int i = 0; i < look_up_table.rows; i++) {
        uchar val(look_up_table.at<uchar>(i));
        if (val > maxVal) {
            maxVal = val;
        }
    }
    for (int i = 0; i < 256; i++) {
        circle(graph, cv::Point{ static_cast<int>(i * kw), static_cast<int>(512. * (1.0 - look_up_table.at<uchar>(i) * 1.0 / maxVal)) }, 1, cv::Scalar(0), cv::FILLED);
    }
    return graph;
}

int main() {
    cv::Mat img = cv::imread("..\\..\\..\\data\\cross_0256x0256.png");
    cv::imwrite("lab03_rgb.png", img);
    cv::Mat look_up_table(create_LUT(img));
    cv::Mat img_res(cv::Mat::zeros(img.size(), img.type()));
    cv::LUT(img, look_up_table, img_res);
    cv::imshow("lab03_rgb_res", img_res);
    cv::imwrite("lab03_rgb_res.png", img_res);

    cv::Mat gray_img;
    cv::cvtColor(img, gray_img, cv::COLOR_BGR2GRAY);
    cv::imshow("lab03_gre", gray_img);
    cv::imwrite("lab03_gre.png", gray_img);

    cv::Mat gray_img_res(cv::Mat::zeros(gray_img.size(), gray_img.type()));
    cv::LUT(gray_img, look_up_table, gray_img_res);
    cv::imshow("lab03_gre_res", gray_img_res);
    cv::imwrite("lab03_gre_res.png", gray_img_res);

    cv::Mat graph(func_graph(look_up_table));
    cv::imshow("lab03_viz_func", graph);
    cv::imwrite("lab03_viz_func.png", graph);
    return 0;
}
```
