#include <jni.h>

#include <android/log.h>
#include <android/bitmap.h>

#include "opencv2/imgproc.hpp"

#define  LOG_TAG    "ContoursDemo"
#define  LOGI(...)  __android_log_print(ANDROID_LOG_INFO,LOG_TAG,__VA_ARGS__)



#ifdef __cplusplus
extern "C" {
#endif





JNIEXPORT void JNICALL
Java_ph_edu_dlsu_contoursdemo_CameraActivity_process(JNIEnv *env, jobject instance,
                                                            jobject pTarget, jbyteArray pSource, jint levels) {
    uint32_t t;
    cv::Mat srcBGR;
    cv::Mat edges;
    std::vector<std::vector<cv::Point> > contours;
    std::vector<cv::Vec4i> hierarchy;

	cv::RNG rng(12345); // for random color
    cv::Scalar color;

    AndroidBitmapInfo bitmapInfo;
    uint32_t* bitmapContent;

    if(AndroidBitmap_getInfo(env, pTarget, &bitmapInfo) < 0) abort();
    if(bitmapInfo.format != ANDROID_BITMAP_FORMAT_RGBA_8888) abort();
    if(AndroidBitmap_lockPixels(env, pTarget, (void**)&bitmapContent) < 0) abort();

    /// Access source array data... OK
    jbyte* source = (jbyte*)env->GetPrimitiveArrayCritical(pSource, 0);
    if (source == NULL) abort();

    /// cv::Mat for YUV420sp source and output BGRA
    cv::Mat src(bitmapInfo.height + bitmapInfo.height/2, bitmapInfo.width, CV_8UC1, (unsigned char *)source);
    cv::Mat mbgra(bitmapInfo.height, bitmapInfo.width, CV_8UC4, (unsigned char *)bitmapContent);


/***********************************************************************************************/
    /// Native Image Processing HERE...

    t = cv::getTickCount();

    if(srcBGR.empty())
        srcBGR = cv::Mat(bitmapInfo.height, bitmapInfo.width, CV_8UC3);

    if (edges.empty())
        edges = cv::Mat(bitmapInfo.height, bitmapInfo.width, CV_8UC1);

    cv::cvtColor(src, srcBGR, CV_YUV420sp2RGB);

    // Detect the edges
    cv::Canny(srcBGR, edges, 50, 100);

    // Find contours
    cv::findContours(edges, contours, hierarchy, CV_RETR_TREE, CV_CHAIN_APPROX_NONE);   


    // Draw contours
    for( size_t i = 0; i < contours.size(); i++ )
    {
        if(contours[i].size() > 50){
        	// Randomize color
            color = cv::Scalar( rng.uniform(0, 255), rng.uniform(0,255), rng.uniform(0,255) );
            drawContours( srcBGR, contours, i, color, 2, 8, hierarchy, levels, cv::Point() );
        }
    }


    LOGI("Processing took %0.2f ms.", 1000*((float)cv::getTickCount() - t)/(float)cv::getTickFrequency());

    cvtColor(srcBGR, mbgra, CV_BGR2BGRA);


/************************************************************************************************/

    /// Release Java byte buffer and unlock backing bitmap
    //env-> ReleasePrimitiveArrayCritical(pSource,source,0);
    /*
     * If 0, then JNI should copy the modified array back into the initial Java
     * array and tell JNI to release its temporary memory buffer.
     *
     * */

    env-> ReleasePrimitiveArrayCritical(pSource, source, JNI_COMMIT);
    /*
 * If JNI_COMMIT, then JNI should copy the modified array back into the
 * initial array but without releasing the memory. That way, the client code
 * can transmit the result back to Java while still pursuing its work on the
 * memory buffer
 *
 * */

    /*
     * Get<Primitive>ArrayCritical() and Release<Primitive>ArrayCritical()
     * are similar to Get<Primitive>ArrayElements() and Release<Primitive>ArrayElements()
     * but are only available to provide a direct access to the target array
     * (instead of a copy). In exchange, the caller must not perform blocking
     * or JNI calls and should not hold the array for a long time
     *
     */


    if (AndroidBitmap_unlockPixels(env, pTarget) < 0) abort();
}


#ifdef __cplusplus
}
#endif
