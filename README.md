# Fourier images

Inspired by 3Blue1Brown video: [https://www.youtube.com/watch?v=r6sGWTCMz2k](https://www.youtube.com/watch?v=r6sGWTCMz2k).

There is a circle and a point on it. The point runs on the circle. The frequency has each circle different (circles per time period). Every circle has it's coefficient expressed by a complex number. The coefficient tells us the starting angle of the point and the radius. You can imagine the complex number written in the trigonometric form. Every running point (except the last) is the center of another circle. The final point is the sum of all points.

The program renderes running points (and their sum) in the time. The result in a time creates a curve. The program allows you to create points and the program will compute coefficients, so the final curve goes through these points. The computation is by Fourier transformation.

You can try it in the browser: [https://fourier-images.pages.dev/](https://fourier-images.pages.dev/). Enjoy.