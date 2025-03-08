 Tech-Event
# **Open cv image read code**
```
import cv2
img =cv2.imread("img.png")
cv2.imshow("fdkvkd",img)
```

## **to create wait box delay**
```
cv2.waitKey(0)
```
## capture video
````
import cv2
vid = cv2.VideoCapture(0)

while True:
    ret, video=vid.read() #ret is booleean function
    cv2.imshow("box", video)
    if cv2.waitKey(1) == ord("a"):
        break

vid.release()
cv2.destroyAllWindows()

````
