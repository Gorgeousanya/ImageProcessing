## Работа 4. Использование бинаризации для анализа изображений (выделение символов)
автор: Плохотнюк А. Д.
дата: 2022-03-21T11:24:08

url: https://github.com/Gorgeousanya/ImageProcessing

### Задание
0. текст, иллюстрации и подписи отчета придумываем самостоятельно
1. самостоятельно снимаем видео смартфоном
- объект съемки - купюры (рубли разного номинала), расправленные и лежащие на поверхности (проективно искаженны прямоугольник)
- количество роликов - от 5 шт.
- длительность - 5-7 сек
- условия съемки разные
2. извлекаем по 3 кадра из каждого ролика (делим кол-во кадров на 5 и берем каждый с индеком 2/5,3/5,4/5)
3. цветоредуцируем изображения
4. бинаризцем изображения
5. морфологически обрабатываем изображения
6. выделяем основную компоненту связности
7. руками изготавливаем маски (идеальная зона купюры)
8. оцениваем качество выделение зоны и анализируем ошибки
### Результаты

![](lab04.src.jpg)
Рис. 1. Исходное тестовое изображение

![](lab04.g1.png)
Рис. 2. Визуализация результата $G_1$ цветоредукции

![](lab04.b1.png)
Рис. 3. Визуализация результата бинаризаии $B_1$

![](lab04.f1.png)
Рис. 4. Визуализация результата $F_1$ фильтрации бинаризованного изображения $B_1$

![](lab04.v1.png)
Рис. 5. Визуализация результатов фильтрации компонент связности $V_1$

![](lab04.e1.png)
Рис. 6. Визуализация отклонений от эталона $E_1$

![](lab04.b2.png)
Рис. 7. Визуализация результата бинаризаии $B_2$ (метод с гауссовым окном)

![](lab04.f2.png)
Рис. 8. Визуализация результата $F_2$ фильтрации бинаризованного изображения $B_2$

![](lab04.v2.png)
Рис. 9. Визуализация результатов фильтрации компонент связности $V_2$ (метод с гауссовым окном)

![](lab04.e2.png)
Рис. 10. Визуализация отклонений от эталона $E_2$ (метод с гауссовым окном)

### Текст программы

```cpp
#include <opencv2/opencv.hpp>

std::vector<cv::Mat> frame(cv::VideoCapture cap, int index) {
    int length = cap.get(cv::CAP_PROP_FRAME_COUNT);
    std::vector<cv::Mat> img;
    cv::Mat image;
    for (int i; i<length; i++){
        if (i == int(length*2/5)){
            cap>>image;
            std::cout<<"lab04_"+std::to_string(index)+"_"+std::to_string(1)+".png";
            cv::waitKey();
            cv::imwrite("lab04_"+std::to_string(index)+"_"+std::to_string(1)+".png", image);
        }
        if (i == int(length*3/5)) {
            cap >> img[1];
            cv::imwrite("lab04_"+std::to_string(index)+"_"+std::to_string(2)+".png", img[1]);
        }
        if (i == int(length*4/5)) {
            cap >> img[2];
            cv::imwrite("lab04_"+std::to_string(index)+"_"+std::to_string(3)+".png", img[2]);
        }
    }
    return img;
}

double quality(cv::Mat detected, cv::Mat correct) {
    double equal = 0;
    double total = 0;

    for (auto i = 0; i < detected.cols; i++) {
        for (auto j = 0; j < detected.rows; j++) {
            if (detected.at<uchar>(j, i) == 255 || correct.at<uchar>(j, i) == 255) {
                total++;
            }
            if (detected.at<uchar>(j, i) == 255 && correct.at<uchar>(j, i) == 255) {
                equal++;
            }
        }
    }

    return equal / total;
}

cv::Mat results(cv::VideoCapture cap, int index){
    std::vector<cv::Mat> img;
    img = frame(cap, index);
    for (int i=0; i<3; i++){
        cv::Mat gray_img, labels, stats, img_edge, centroids, img_color, bi, filt, morph, image;
        //сжатие изображения
        cv::imwrite("lab04.src.png", img[i], {cv::IMWRITE_PNG_COMPRESSION, 5} );
        img[i] = cv::imread("/Users/annaplokhotnyuk/Downloads/plokhotnyuk_a_d/bin.dbg/lab04.src.jpg");
        //цветоредуцирование
        cv::cvtColor(img[i], gray_img, cv::COLOR_BGR2GRAY);
        cv::imwrite("gray.png" , gray_img);
        //бинаризация
        cv::adaptiveThreshold(gray_img, bi, 255, cv::ADAPTIVE_THRESH_GAUSSIAN_C, cv::THRESH_BINARY, 71, 13);
        cv::imwrite("bi.png" , bi);
        //морфология
        cv::morphologyEx(bi, morph, cv::MORPH_OPEN, cv::getStructuringElement(cv::MORPH_RECT, cv::Size(5, 5)));
        cv::morphologyEx(morph, morph, cv::MORPH_CLOSE, cv::getStructuringElement(cv::MORPH_RECT, cv::Size(5, 5)));

        //фильтрация
        //cv::medianBlur(bi, filt, 3);
        //cv::imwrite("filter.png" , filt);

        cv::hconcat(img[i], gray_img, image);
        cv::hconcat(image, bi, image);

        //выделение компонент связности
        cv::connectedComponentsWithStats(morph, labels, stats, centroids);

        //std::cout << labels << std::endl;
        std::cout << "stats.size()=" << stats.size() << std::endl;
        //std::cout << centroids << std::endl;

        for(int i=0; i<stats.rows; i++)
        {
            int x = stats.at<int>(cv::Point(0, i));
            int y = stats.at<int>(cv::Point(1, i));
            int w = stats.at<int>(cv::Point(2, i));
            int h = stats.at<int>(cv::Point(3, i));

            std::cout << "x=" << x << " y=" << y << " w=" << w << " h=" << h << std::endl;

            cv::Scalar color(255,255,255);
            cv::Rect rect(x,y,w,h);
            cv::rectangle(img, rect, color, -1);
        }
        cv::imwrite("img.png", img[0]);

        //шаблонная маска

        //коллаж из изображений

        cv::hconcat(image, img[0], image);
        //оценка результатов
        std::cout<<"Результаты сравнения качества для"<<index<<"видео:"<<quality(img[0], img[1])<<'/n';

        return image;
    }

}

int main() {
    std::vector<cv::Mat> src;
    cv::Mat image;
    cv::VideoCapture cap("/Users/annaplokhotnyuk/Downloads/plokhotnyuk_a_d/data/IMG_1.mov");
    src = frame(cap, 1);
    cv::VideoCapture cap2("/Users/annaplokhotnyuk/Downloads/plokhotnyuk_a_d/data/IMG_2.mov");
    src = frame(cap2, 2);
    cv::VideoCapture cap3("/Users/annaplokhotnyuk/Downloads/plokhotnyuk_a_d/data/IMG_3.mov");
    src = frame(cap3, 3);
    cv::VideoCapture cap4("/Users/annaplokhotnyuk/Downloads/plokhotnyuk_a_d/data/IMG_4.mov");
    src = frame(cap4, 4);
    cv::VideoCapture cap5("/Users/annaplokhotnyuk/Downloads/plokhotnyuk_a_d/data/IMG_5.mov");
    src = frame(cap5, 5);

    return 0;
}
```
