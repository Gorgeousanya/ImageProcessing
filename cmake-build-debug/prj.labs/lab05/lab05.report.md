## Работа 5. Детектирование границ документов на кадрах видео
автор: Плохотнюк А. Д.
дата: 2022-04-10T19:45:21

url: https://github.com/Gorgeousanya/ImageProcessing

### Задание
0. текст, иллюстрации и подписи отчета придумываем самостоятельно
1. используем данные из лаб. 4
2. выделяем границы и находим внешнюю рамку документа
3. руками изготавливаем векторную разметку (идеальная рамка купюры)
4. оцениваем качество и анализируем ошибки

### Результаты

### Текст программы

```cpp
#include <opencv2/opencv.hpp>

cv::Mat src_gray;
int thresh = 50;
cv::RNG rng(12345);
void thresh_callback(int, void* );

void frame(cv::VideoCapture cap, int index) {
    long length = cap.get(cv::CAP_PROP_FRAME_COUNT);
    cv::Mat image1, image2, image3, image;
    for (int i=0; i<length; i++){
        if (i == long((length*2)/5)){
            cap>>image1;
            cv::imwrite("lab04_"+std::to_string(index)+"_"+std::to_string(1)+".png", image1);
        } else
        if (i == long((length*3)/5)) {
            cap>>image2;
            cv::imwrite("lab04_"+std::to_string(index)+"_"+std::to_string(2)+".png", image2);
        } else
        if (i == long((length*4)/5)) {
            cap>>image3;
            cv::imwrite("lab04_"+std::to_string(index)+"_"+std::to_string(3)+".png", image3 );
        } else {
            cap>>image;
        }
    }
}

double quality(cv::Mat detected, cv::Mat correct) {
    double equal = 0., total = 0.;
    for (int i = 0; i < detected.cols; i++) {
        for (int j = 0; j < detected.rows; j++) {
            if (detected.at<uchar>(j, i) != 0 || correct.at<uchar>(j, i) != 0) {
                total++;
            }
            if (detected.at<uchar>(j, i) != 0 && correct.at<uchar>(j, i) != 0) {
                equal++;
            }
        }
    }
    return equal / total;
}

void thresh_callback(int, void* )
{
    cv::Mat canny_output;
    Canny( src_gray, canny_output, thresh, thresh*2 );
    std::vector<std::vector<cv::Point> > contours;
    std::vector<cv::Vec4i> hierarchy;
    findContours( canny_output, contours, hierarchy, cv::RETR_EXTERNAL, cv::CHAIN_APPROX_SIMPLE );
    cv::Mat drawing = cv::Mat::zeros( canny_output.size(), CV_8UC3 );
    for( size_t i = 0; i< contours.size(); i++ )
    {
        cv::Scalar color = cv::Scalar( rng.uniform(0, 256), rng.uniform(0,256), rng.uniform(0,256) );
        drawContours( drawing, contours, (int)i, color, 2, cv::LINE_8, hierarchy, 0 );
    }
    imshow( "Contours", drawing );
}

cv::Mat final(cv::Mat img, cv::Mat detected, cv::Mat cor) {
    cv::Mat rgbImage, rgbImageChannels[3];
    cv::cvtColor(img, rgbImage, cv::COLOR_GRAY2BGR);
    cv::split(rgbImage, rgbImageChannels);
    cv::max(rgbImageChannels[2], detected, rgbImageChannels[2]);
    cv::max(rgbImageChannels[0], cor, rgbImageChannels[0]);
    cv::merge(rgbImageChannels, 3,  rgbImage);
    return rgbImage;
}

void mask(cv::Mat& img, std::string filename){
    cv::FileStorage fs("/Users/annaplokhotnyuk/Downloads/plokhotnyuk_a_d/data/correct.json", {cv::FileStorage::READ | cv::FileStorage::FORMAT_JSON});
    cv::FileNode x = fs[filename]["x"];
    cv::FileNode y = fs[filename]["y"];
    img = cv::Scalar (0);
    std::vector<cv::Point>point;
    for (int i = 0; i < x.size(); i++) {
        point.push_back(cv::Point(x[i], y[i]));
    }
    cv::fillPoly(img, point, cv::Scalar(255, 255, 255));
    cv::imwrite("mask"+filename, img);
}


void results(cv::VideoCapture cap, int index){
    std::vector<cv::Mat> img;
    frame(cap, index);
    for (int i=0; i<3; i++){
        cv::Mat src=cv::imread("/Users/annaplokhotnyuk/Downloads/plokhotnyuk_a_d/bin.dbg/lab04_"+std::to_string(index)+"_"+std::to_string(i+1)+".png");
        img.push_back(src);
        cv::Mat gray_img, labels, stats, img_edge, centroids, img_color, bi, filt, morph, image;

        //цветоредуцирование
        cv::cvtColor(img[i], gray_img, cv::COLOR_BGR2GRAY);
        cv::imwrite("gray.png" , gray_img);

        //бинаризация
        cv::adaptiveThreshold(gray_img, bi, 255, cv::ADAPTIVE_THRESH_GAUSSIAN_C, cv::THRESH_BINARY, 71, 9);
        cv::imwrite("bi.png" , bi);
        cv::medianBlur(bi, filt, 3);

        //морфология
//        cv::Mat kernel = cv::getStructuringElement(cv::MORPH_RECT, cv::Size((2 * 2) + 1,(3 * 3) + 1));
//        cv::erode(bi, morph, kernel, cv::Point(-1, -1));
//        cv::dilate(morph, morph, kernel);
//        cv::morphologyEx(morph, morph, cv::MORPH_OPEN, cv::getStructuringElement(cv::MORPH_RECT, cv::Size(3, 3)));
//        cv::morphologyEx(morph, morph, cv::MORPH_CLOSE, cv::getStructuringElement(cv::MORPH_RECT, cv::Size(3, 3)));


        //find contours
        cv::Mat canny_output;
        Canny( gray_img, canny_output, thresh, thresh*1.5 );
        std::vector<std::vector<cv::Point> > contours;
        std::vector<cv::Vec4i> hierarchy;
        findContours(bi, contours, hierarchy, cv::RETR_TREE, cv::CHAIN_APPROX_NONE );
        cv::Mat drawing = cv::Mat::zeros( bi.size(), CV_8UC3 );
        //for (int i=0; i< hierarchy.size(); i++)
        //std::cout<<hierarchy[i];
//        for( size_t i = 2; i< contours.size(); i++ )
//        {
//            cv::Scalar color = cv::Scalar( 255, 255, 255 );
//            drawContours( drawing, contours, (int)i, color, 2, cv::LINE_8, hierarchy, 0 );
//        }
        //imshow( "Contours", drawing );
        cv::drawContours(cv::imread(
                    "/Users/annaplokhotnyuk/Downloads/plokhotnyuk_a_d/bin.dbg/lab04_" + std::to_string(index) + "_" +
                    std::to_string(i+1) + ".png"), contours, -1, cv::Scalar(0, 255, 0), 2);
        cv::imwrite("contour"+std::to_string(index)+"_"+std::to_string(i+1)+".png", img[i]);
    }
}

int main() {
    cv::Mat src, mask;
    for (int i=0; i<5; i++) {
        cv::VideoCapture cap("/Users/annaplokhotnyuk/Downloads/plokhotnyuk_a_d/data/IMG_"+std::to_string(i+1)+".mov");
        results(cap, i+1);
    }
//    for (int i=0; i<5; i++) {
//        cv::Mat image, image_mask;
//        for (int j = 0; j < 3; j++) {
//            image.push_back(cv::imread(
//                    "/Users/annaplokhotnyuk/Downloads/plokhotnyuk_a_d/bin.dbg/lab04_" + std::to_string(i + 1) + "_" +
//                    std::to_string(j + 1) + ".png"));
//            image_mask.push_back(cv::imread(
//                    "/Users/annaplokhotnyuk/Downloads/plokhotnyuk_a_d/bin.dbg/final" + std::to_string(i + 1) + "_" +
//                    std::to_string(j + 1) + ".png"));
//        }
//        if (!src.empty()) {
//            cv::hconcat(src, image, src);
//            cv::hconcat(mask, image_mask, mask);
//        } else {
//            src = image.clone();
//            mask = image_mask.clone();
//        }
//    }
//    cv::imwrite("src.png", src);
//    cv::imwrite("result.png", mask);
    return 0;
}
```
