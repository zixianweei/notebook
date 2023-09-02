

=== "C++"

    ```cpp
    #include <iostream>
    #include <limits>
    #include <numeric>
    #include <queue>
    #include <utility>
    #include <vector>
    
    #include <opencv2/core.hpp>
    #include <opencv2/highgui.hpp>
    #include <opencv2/imgcodecs.hpp>
    #include <opencv2/imgproc.hpp>
    
    #define NOTREACHED() std::cout << "NOTREACHED\n";
    
    enum class Metrics : uint8_t {
      kL1 = 0,
      kL2 = 1,
      kBoundary = std::numeric_limits<uint8_t>::max(),
    };
    
    class Object {
     public:
      Object(int x, int y, int c) : x_(x), y_(y), c_(c) {}
    
      [[nodiscard]] int x() const { return x_; }
      [[nodiscard]] int y() const { return y_; }
      [[nodiscard]] int c() const { return c_; }
    
      [[nodiscard]] double distance(int x, int y, Metrics metrics = Metrics::kL1) const {
        assert(metrics < Metrics::kBoundary && "Metrics out of boundary.");
        if (metrics == Metrics::kL1) {
          return abs(x - this->x()) + abs(y - this->y());
        }
        if (metrics == Metrics::kL2) {
          double pdist = std::pow(abs(x - this->x()), 2.0) + std::pow(abs(y - this->y()), 2.0);
          return std::sqrt(pdist);
        }
    
        NOTREACHED();
        return 0.0;
      }
    
     private:
      int x_;
      int y_;
      int c_;
    };
    
    void knn(cv::Mat& canvas,
             int kClose,
             const std::vector<Object>& objects,
             int numOfClasses,
             Metrics metrics,
             const std::vector<cv::Scalar>& randColors,
             double maxDistance) {
      auto compare = [](const auto& p1, const auto& p2) { return p1.first < p2.first; };
    
      // 逐个计算像素中每个点和每个object之间的距离
      int rows = canvas.rows;
      int cols = canvas.cols;
      for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
          std::priority_queue<std::pair<double, Object>, std::vector<std::pair<double, Object>>, decltype(compare)> maxHeap(
              compare);
          for (const auto& o : objects) {
            auto dist = o.distance(c, r, metrics);
            if (dist >= maxDistance) {
              continue;
            }
            maxHeap.emplace(dist, o);
            if (maxHeap.size() > kClose) {
              maxHeap.pop();
            }
          }
          if (!maxHeap.empty()) {
            std::vector<int> voter(numOfClasses, 0);
            while (!maxHeap.empty()) {
              auto o = maxHeap.top();
              maxHeap.pop();
              voter[o.second.c()]++;
            }
    
            auto most = std::max_element(voter.begin(), voter.end()) - voter.begin();
            canvas.at<cv::Vec4b>(cv::Point(c, r)) = randColors[most];
          }
        }
      }
    }
    
    int main() {
      int numOfClasses = 4;
      int numOfPoints = 40;
      int clusterDistance = 50;
      Metrics metrics = Metrics::kL1;
      int kClose = 5;
      int canvasSize = 600;
      double maxDistance = canvasSize >> 1;
    
      cv::Mat canvas = cv::Mat::zeros(cv::Size(canvasSize, canvasSize), CV_8UC4);
    
      std::vector<Object> objects;
      objects.reserve(numOfPoints);
    
      cv::RNG rng(cv::getTickCount());
    
      // 生成随机颜色 每种类别对应一种随机颜色
      std::vector<cv::Scalar> randColors;
      randColors.reserve(numOfClasses);
      for (int i = 0; i < numOfClasses; i++) {
        randColors.emplace_back(rng.uniform(0, 255), rng.uniform(0, 255), rng.uniform(0, 255), 191);
      }
    
      // 生成随机种子和种子簇
      for (int i = 0; i < numOfClasses; i++) {
        objects.emplace_back(rng.uniform(0, canvasSize), rng.uniform(0, canvasSize), i);
      }
      for (int i = numOfClasses; i < numOfPoints; i++) {
        int x = objects[i % numOfClasses].x() + rng.uniform(-clusterDistance, clusterDistance);
        int y = objects[i % numOfClasses].y() + rng.uniform(-clusterDistance, clusterDistance);
        int c = objects[i % numOfClasses].c();
        objects.emplace_back(x, y, c);
      }
    
      knn(canvas, kClose, objects, numOfClasses, metrics, randColors, maxDistance);
    
      // 后处理 乘以透明度 然后将所有种子点绘制上去
      for (int i = 0; i < numOfPoints; i++) {
        auto color = randColors[objects[i].c()] * 0.75;
        color[3] = 255;
        cv::circle(canvas, cv::Point(objects[i].x(), objects[i].y()), 5, color, -1);
      }
    
      cv::imwrite("knn.png", canvas);
    
      return 0;
    }
    ```
