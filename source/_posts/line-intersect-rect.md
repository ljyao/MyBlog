---
title: 判断直线与矩形相交
date: 2017-03-07 10:15:18
tags:
---
```
public class Rectangle {
    public static final int OUT_LEFT = 1;

    public static final int OUT_TOP = 2;

    public static final int OUT_RIGHT = 4;

    public static final int OUT_BOTTOM = 8;
    private float width;
    private float height;
    private float left;
    private float top;

    public Rectangle(float x1, float y1, float w, float h) {
        this.left = x1;
        this.top = y1;
        width = w;
        height = h;
    }

    public boolean intersectsLine(float x1, float y1, float x2, float y2) {
        int out1, out2;
        if ((out2 = outcode(x2, y2)) == 0) {
            return true;
        }
        while ((out1 = outcode(x1, y1)) != 0) {
            if ((out1 & out2) != 0) {
                return false;
            }
            if ((out1 & (OUT_LEFT | OUT_RIGHT)) != 0) {
                float x = left;
                if ((out1 & OUT_RIGHT) != 0) {
                    x += width;
                }
                y1 = y1 + (x - x1) * (y2 - y1) / (x2 - x1);
                x1 = x;
            } else {
                float y = top;
                if ((out1 & OUT_BOTTOM) != 0) {
                    y += height;
                }
                x1 = x1 + (y - y1) * (x2 - x1) / (y2 - y1);
                y1 = y;
            }
        }
        return true;
    }

    public int outcode(double x, double y) {
        int out = 0;
        if (this.width <= 0) {
            out |= OUT_LEFT | OUT_RIGHT;
        } else if (x < this.left) {
            out |= OUT_LEFT;
        } else if (x > this.left + this.width) {
            out |= OUT_RIGHT;
        }
        if (this.height <= 0) {
            out |= OUT_TOP | OUT_BOTTOM;
        } else if (y < this.top) {
            out |= OUT_TOP;
        } else if (y > this.top + this.height) {
            out |= OUT_BOTTOM;
        }
        return out;
    }
}
```