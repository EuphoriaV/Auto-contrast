#include <iostream>
#include"omp.h"
#include<vector>
#include<chrono>
using namespace std;

/*
@author Kuznetsov Ilya ilyakuznecov84@gmail.com
*/

/*
argv[1] number of threads
argv[2] name of file
argv[3] name of new file
argv[4] contrast coefficient
*/

int main(int argc, char* argv[]) {
    chrono::high_resolution_clock::time_point start = chrono::high_resolution_clock::now();
    if (argc != 5) {
        cout << "Wrong arguments format";
        return -1;
    }
    int numOfThreads = atoi(argv[1]);
    if (numOfThreads < 0) {
        cout << "Input number of threads >= 0";
        return -1;
    }
    omp_set_num_threads(numOfThreads);
    string first = argv[2];
    if (!(first.compare(first.size() - 4, 4, ".ppm") == 0 || first.compare(first.size() - 4, 4, ".pgm") == 0 || first.compare(first.size() - 4, 4, ".pnm") == 0)) {
        cout << "Files must be .ppm or .pgm or .pnm";
        return -1;
    }
    string second = argv[3];
    if (!(second.compare(second.size() - 4, 4, ".ppm") == 0 || second.compare(second.size() - 4, 4, ".pgm") == 0 || second.compare(second.size() - 4, 4, ".pnm") == 0)) {
        cout << "Files must be .ppm or .pgm or .pnm";
        return -1;
    }
    float k = atof(argv[4]);
    FILE* in = fopen(argv[2], "rb");
    FILE* out = fopen(argv[3], "wb");
    if (in == NULL) {
        cout << "File is not found";
        return -1;
    }
    bool isPpm;
    char p;
    int width, height, value;
    fscanf(in, "P%c%d%d%d\n", &p, &width, &height, &value);
    if (p == '6')isPpm = true;
    else if (p == '5')isPpm = false;
    else return -1;
    unsigned char* file;
    if (isPpm) file = new unsigned char[3 * width * height];
    else file = new unsigned char[width * height];
    if (value != 255) {
        cout << "max value must be 255";
        return -1;
    }
    //
    //ppm
    //
    if (isPpm) {
        int red[256], green[256], blue[256];
        for (int i = 0; i < 256; i++) {
            red[i] = 0;
            green[i] = 0;
            blue[i] = 0;
        }
        fread(file, 1, height * width * 3, in);
#pragma omp parallel for schedule (static)
        for (int i = 0; i < height * width * 3; i += 3) {
            red[(int)file[i]]++;
            green[(int)file[i + 1]]++;
            blue[(int)file[i + 2]]++;
        }
        int minR = 255, minG = 255, minB = 255, sumR = 0, sumG = 0, sumB = 0, maxR = 0, maxG = 0, maxB = 0;
        for (int i = 0; i < 256; i++) {
            sumR += red[i];
            sumG += green[i];
            sumB += blue[i];
            if (sumR >= height * width * k && minR == 255) {
                minR = i;
            }
            if (sumG >= height * width * k && minG == 255) {
                minG = i;
            }
            if (sumB >= height * width * k && minB == 255) {
                minB = i;
            }
        }
        sumR = 0;
        sumG = 0;
        sumB = 0;
        for (int i = 255; i >= 0; i--) {
            sumR += red[i];
            sumG += green[i];
            sumB += blue[i];
            if (sumR >= height * width * k && maxR == 0) {
                maxR = i;
            }
            if (sumG >= height * width * k && maxG == 0) {
                maxG = i;
            }
            if (sumB >= height * width * k && maxB == 0) {
                maxB = i;
            }
        }
        fprintf(out, "P6\n%d %d\n255\n", width, height);
#pragma omp parallel for schedule (static)
        for (int i = 0; i < height * width * 3; i += 3) {
            int r = file[i];
            int g = file[i + 1];
            int b = file[i + 2];
            if (r <= minR) {
                file[i] = (unsigned char)(0);
            }
            else if (r >= maxR) {
                file[i] = (unsigned char)(255);
            }
            else {
                file[i] = (unsigned char)(((r - minR) * 255) / (maxR - minR));
            }
            if (g <= minG) {
                file[i + 1] = (unsigned char)(0);
            }
            else if (g >= maxG) {
                file[i + 1] = (unsigned char)(255);
            }
            else {
                file[i + 1] = (unsigned char)(((g - minG) * 255) / (maxG - minG));
            }
            if (b <= minB) {
                file[i + 2] = (unsigned char)(0);
            }
            else if (b >= maxB) {
                file[i + 2] = (unsigned char)255;
            }
            else {
                file[i + 2] = (unsigned char)(((b - minB) * 255) / (maxB - minB));
            }
        }
        fwrite(file, width * height * 3, 1, out);
    }
    //
    //pgm
    //
    else {
        int mas[256];
        for (int i = 0; i < 256; i++) {
            mas[i] = 0;
        }
        fread(file, 1, height * width, in);
#pragma omp parallel for schedule (static)
        for (int i = 0; i < height * width; i++) {
            mas[(int)file[i]]++;
        }
        int min, sum = 0, max;
        for (int i = 0; i < 256; i++) {
            sum += mas[i];
            if (sum >= height * width * k) {
                min = i;
                break;
            }
        }
        sum = 0;
        for (int i = 255; i >= 0; i--) {
            sum += mas[i];
            if (sum >= height * width * k) {
                max = i;
                break;
            }
        }
        fprintf(out, "P5\n%d %d\n255\n", width, height);
#pragma omp parallel for schedule (static)
        for (int i = 0; i < width * height; i++) {
            int x = file[i];
            if (x <= min) {
                file[i] = (unsigned char)(0);
            }
            else if (x >= max) {
                file[i] = (unsigned char)(255);
            }
            else {
                file[i] = (unsigned char)(((x - min) * 255) / (max - min));
            }
        }
        fwrite(file, width * height, 1, out);
    }
    fclose(in);
    fclose(out);
    delete[] file;
    chrono::high_resolution_clock::time_point end = chrono::high_resolution_clock::now();
    chrono::duration<double, std::milli> time = end - start;
    printf("Time(%i thread(s)) : %g ms\n", numOfThreads, time.count());
    return 0;
}
