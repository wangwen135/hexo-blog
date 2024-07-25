---
title: java画监控面板
date: 2019-08-02 23:41
tags: 
  - java-swing
categories:
  - [java-swing]
---

![监控界面.png](https://upload-images.jianshu.io/upload_images/2043910-8ba19df71a9d6802.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 画面板 MonitorPanel
```
package com.wwh.test.swing.monitor;

import java.awt.Color;
import java.awt.Font;
import java.awt.FontMetrics;
import java.awt.Graphics;
import java.awt.Graphics2D;
import java.awt.image.BufferedImage;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

import javax.swing.JPanel;

/**
 * <pre>
 * 有XY轴的曲线图监控面板
 * 
 * <pre>
 * 
 * @author 313921
 * @date 2015-01-28 22:11:04
 */
public class MonitorPanel extends JPanel implements MonitorPanelModelListener {

    /**
     * 最合适的Y轴线条高度间距
     */
    private static final int bastYAxisLineHeightInterval = 26;

    /**
     * X轴上默认的线与线之间的像素
     */
    public static final int DEFAULT_X_AXIS_LINE_INTERVAL = 70;
    /**
     * 默认的当前值文本宽度
     */
    private static final int defaultCurrentTextWidth = 50;

    /**
     * 最少的Y轴线条数量
     */
    private static final int minYAxisLineCount = 3;

    private static final long serialVersionUID = 1L;

    /**
     * 图片
     */
    private BufferedImage bufferImage;

    /**
     * 显示当前值的文本宽度
     */
    private int currentTextWidth = 50;

    private MonitorPanelModel dataModel;

    private SimpleDateFormat dateF = new SimpleDateFormat("HH:mm");

    /**
     * 是否显示最后一个值
     */
    private boolean displayLastValue = true;

    private boolean drawEmptyGraph = true;

    private Color fontColor = Color.BLACK;

    /**
     * 曲线区域的背景颜色
     */
    private Color graphBackgroundColor = new Color(0, 100, 0);

    private Graphics2D graphics;

    /**
     * 曲线的颜色
     */
    private Color graphLineColor = new Color(250, 0, 0);

    private String graphTitle = "监控面板";

    private Color gridLineColor = new Color(0, 0, 0);
    /**
     * 图片的高度
     */
    private int imgHeight;

    /**
     * 图片的宽度，整个组件就是一个画上去的图片
     */
    private int imgWidth;

    /**
     * 标题字体
     */
    private Font titleFont = new Font("宋体", Font.BOLD, 14);

    /**
     * 标题颜色
     */
    private Color titleFontColor = Color.BLACK;

    /**
     * 标题的高度
     */
    private int titleHeight = 20;

    /**
     * X轴上的线与线之间的像素
     */
    private int xAxisLineInterval = DEFAULT_X_AXIS_LINE_INTERVAL;

    /**
     * X轴上的一个与像素点的比例
     */
    private double xRatio;

    /**
     * X轴的文本高度
     */
    private int XTextHeight = 20;

    /**
     * Y轴上一个与像素点的比例
     */
    private double yRatio;

    /**
     * Y轴的文本宽度
     */
    private int YTextWidth = 40;

    public MonitorPanel() {
        this(null);
    }

    public MonitorPanel(MonitorPanelModel model) {
        if (model == null) {
            model = new MonitorPanelModel();
        }
        setModel(model);
        updateUI();
    }

    /**
     * <pre>
     * 计算X轴的比例
     * </pre>
     */
    private void calcXRatio() {
        Long maxXCoords = dataModel.getMaxXCoords();
        if (maxXCoords == null) {
            return;
        }
        Long minXCoords = dataModel.getMinXCoords();
        if (minXCoords == null) {
            return;
        }
        // 曲线图的宽度
        int graphWidth = getGraphWidth();
        if (graphWidth <= 0) {
            xRatio = 0;
            return;
        }
        long xv = maxXCoords - minXCoords;
        // 得到比例
        xRatio = (double) graphWidth / (double) xv;
    }

    /**
     * <pre>
     * 计算Y轴比例 
     * 既：值与显示区域的像素的比例
     * </pre>
     */
    private void calcYRatio() {
        Long maxYCoords = dataModel.getMaxYCoords();
        if (maxYCoords == null) {
            return;
        }
        Long minYCoords = dataModel.getMinYCoords();
        if (minYCoords == null) {
            return;
        }
        // 计算曲线图的高度
        int graphHeight = getGraphHeight();

        if (graphHeight <= 0) {
            yRatio = 0;
            return;
        }
        long yd = maxYCoords - minYCoords;// Y轴
        if (yd <= 0) {
            yRatio = 0;
            return;
        }
        yRatio = (double) graphHeight / (double) yd;// 得到比例
    }

    private void clearBufferImage() {
        graphics.setBackground(getBackground());
        graphics.clearRect(0, 0, imgWidth, imgHeight);
    }

    private void drawDefaultXAxis() {
        int graphWidth = getGraphWidth();
        int graphHeight = getGraphHeight();
        // 每隔一定的像素画一条竖线
        graphics.setColor(gridLineColor);

        for (int i = YTextWidth + xAxisLineInterval; i < YTextWidth + graphWidth; i += xAxisLineInterval) {
            graphics.drawLine(i, titleHeight, i, titleHeight + graphHeight);
        }
    }

    private void drawDefaultYAxis() {
        // 每隔一定的像素画一条线
        int graphWidth = getGraphWidth();
        int graphHeight = getGraphHeight();

        // 画横线
        graphics.setColor(gridLineColor);
        for (int i = graphHeight; i - bastYAxisLineHeightInterval > 0; i -= bastYAxisLineHeightInterval) {
            int yPosition = i - bastYAxisLineHeightInterval + titleHeight;
            graphics.drawLine(YTextWidth, yPosition, YTextWidth + graphWidth, yPosition);
        }
    }

    /**
     * 绘制曲线
     */
    private void drawGraph() {
        graphics.setColor(graphLineColor);
        List<Coordinate> coordinatelist = dataModel.getCoordinatelist();
        int size = coordinatelist.size();
        if (size < 2) {
            return;
        }
        int[] xPoints = new int[size];
        int[] yPoints = new int[size];

        for (int i = 0; i < size; i++) {
            xPoints[i] = getXPoint(coordinatelist.get(i).getxValue());
            yPoints[i] = getYPoint(coordinatelist.get(i).getyValue());
        }

        graphics.drawPolyline(xPoints, yPoints, size);

        // 画一个当前值
        Font font = graphics.getFont();
        FontMetrics fm = graphics.getFontMetrics(font);
        int descent = fm.getDescent();
        graphics.setColor(fontColor);
        int _yPoint = yPoints[size - 1];
        int graphWidth = getGraphWidth();
        int _xPoint = YTextWidth + graphWidth + 1;
        long currentValue = coordinatelist.get(coordinatelist.size() - 1).getyValue();
        String valueOf = String.valueOf(currentValue);
        graphics.drawString(valueOf, _xPoint, _yPoint - descent);
        graphics.drawLine(_xPoint, _yPoint, _xPoint + fm.stringWidth(valueOf), _yPoint);
    }

    /**
     * <pre>
     * 画背景
     * </pre>
     */
    private void drawGraphBackgound() {
        int graphWidth = getGraphWidth();
        int graphHeight = getGraphHeight();

        graphics.setColor(graphBackgroundColor);
        // big.fill3DRect(YTextWidth, titleHeight, imgWidth - YTextWidth
        // - cutlineWidth, imgHeight - titleHeight - XTextHeight, false);

        // 填充指定的矩形。该矩形左边缘和右边缘分别位于 x 和 x + width - 1。上边缘和下边缘分别位于 y 和 y + height -
        // 1。得到的矩形覆盖 width 像素宽乘以 height 像素高的区域。使用图形上下文的当前颜色填充该矩形。
        graphics.fillRect(YTextWidth, titleHeight, graphWidth, graphHeight);

        graphics.setColor(gridLineColor);
        // 绘制指定矩形的边框。矩形的左边缘和右边缘分别位于 x 和 x + width。上边缘和下边缘分别位于 y 和 y +
        // height。使用图形上下文的当前颜色绘制该矩形。
        graphics.drawRect(YTextWidth, titleHeight, graphWidth, graphHeight);

    }

    /**
     * <pre>
     * 将内容画在图片上
     * </pre>
     */
    private void drawImage() {
        int tw = getWidth();
        int th = getHeight();
        if (tw != imgWidth || th != imgHeight) {
            // 大小有变化，重建一张图片
            imgWidth = tw;
            imgHeight = th;
            bufferImage = new BufferedImage(imgWidth, imgHeight, BufferedImage.TYPE_INT_RGB);
            graphics = bufferImage.createGraphics();
        }
        if (bufferImage == null) {
            return;
        }

        // 先清空图片
        clearBufferImage();

        // 画标题
        drawTitle();

        // 画内容区域 背景
        drawGraphBackgound();

        // 获取XY轴上的线
        long[] xLine = dataModel.getXAxisLines(getBestXAxisLineCount());
        long[] yLine = dataModel.getYAxisLines(getBestYAxisLineCount());

        // 计算X 和 Y 的比例
        calcYRatio();
        calcXRatio();

        // 画X坐标
        if (xLine != null) {
            drawXAxis(xLine);
        } else {
            if (drawEmptyGraph) {
                // 画缺省的X坐标
                drawDefaultXAxis();
            }
        }

        // 画Y坐标
        if (yLine != null) {
            drawYAxis(yLine);
        } else if (drawEmptyGraph) {
            // 画缺省的Y坐标
            drawDefaultYAxis();
        }

        // 画曲线
        drawGraph();
    }

    /**
     * <pre>
     * 画标题
     * </pre>
     */
    private void drawTitle() {
        if (graphTitle == null || graphTitle.isEmpty() || titleHeight < 1) {
            return;
        }
        graphics.setColor(titleFontColor);
        graphics.setFont(titleFont);
        FontMetrics fm = graphics.getFontMetrics(titleFont);
        int titleWidth = fm.stringWidth(graphTitle);

        int graphWidth = getGraphWidth();
        int titleX = (graphWidth - titleWidth) / 2 + YTextWidth;
        graphics.drawString(graphTitle, titleX, fm.getAscent());
    }

    /**
     * <pre>
     * 画X轴
     * </pre>
     */
    private void drawXAxis(long[] xLine) {
        if (xLine == null) {
            return;
        }

        Font font = graphics.getFont();
        FontMetrics fm = graphics.getFontMetrics(font);
        int ascent = fm.getAscent();
        // int descent = fm.getDescent();
        // int fHeight = fm.getHeight();

        int graphHeight = getGraphHeight();

        // 画X坐标
        for (long l : xLine) {
            int xPoint = getXPoint(l);

            graphics.setColor(gridLineColor);
            graphics.drawLine(xPoint, titleHeight, xPoint, titleHeight + graphHeight);

            Date date = new Date(l);
            String dateStr = dateF.format(date);

            graphics.setColor(fontColor);
            graphics.drawString(dateStr, xPoint, titleHeight + graphHeight + ascent);

        }
    }

    /**
     * <pre>
     * 画Y轴
     * </pre>
     */
    private void drawYAxis(long[] yLine) {
        if (yLine == null) {
            return;
        }

        Font font = graphics.getFont();
        FontMetrics fm = graphics.getFontMetrics(font);
        int descent = fm.getDescent();
        // 先设置字体
        graphics.setFont(font);

        // 画最大值
        // String maxYStr = String.valueOf(maxY);
        // big.drawString(maxYStr, YTextWidth - fm.stringWidth(maxYStr) - 1,
        // titleHeight + descent);
        // // 画最小值
        // String minYStr = String.valueOf(minY);
        // big.drawString(minYStr, YTextWidth - fm.stringWidth(minYStr) - 1,
        // titleHeight + graphHeight);

        int graphWidth = getGraphWidth();

        for (long l : yLine) {
            String yNumberStr = String.valueOf(l);
            int yPosition = getYPoint(l);
            if (YTextWidth > 0) {
                // 数字
                graphics.setColor(fontColor);
                graphics.drawString(yNumberStr, YTextWidth - fm.stringWidth(yNumberStr) - 1, yPosition + descent);
            }

            // 画横线
            graphics.setColor(gridLineColor);
            graphics.drawLine(YTextWidth, yPosition, YTextWidth + graphWidth, yPosition);

        }
    }

    private int getBestXAxisLineCount() {
        // 在X轴能显示几条线
        int lineCount = getGraphWidth() / xAxisLineInterval;
        lineCount = lineCount > 1 ? lineCount : 1;
        return lineCount;
    }

    /**
     * 根据坐标系高度计算最佳的横线条数
     * 
     * @return
     */
    private int getBestYAxisLineCount() {
        int lineCount = getGraphHeight() / bastYAxisLineHeightInterval;
        lineCount = lineCount > minYAxisLineCount ? lineCount : minYAxisLineCount;
        return lineCount;
    }

    public int getCurrentTextWidth() {
        return currentTextWidth;
    }

    public boolean getDisplayLastValue() {
        return displayLastValue;
    }

    public Color getFontColor() {
        return fontColor;
    }

    public Color getGraphBackgroundColor() {
        return graphBackgroundColor;
    }

    /**
     * 获取曲线图区域的高度
     * 
     * @return
     */
    public int getGraphHeight() {
        if (bufferImage == null) {
            return 0;
        }
        return bufferImage.getHeight() - titleHeight - XTextHeight;
    }

    public Color getGraphLineColor() {
        return graphLineColor;
    }

    public String getGraphTitle() {
        return graphTitle;
    }

    /**
     * 获取曲线图区域的宽度
     * 
     * @return
     */
    public int getGraphWidth() {
        if (bufferImage == null) {
            return 0;
        }
        return bufferImage.getWidth() - YTextWidth - currentTextWidth;
    }

    public Color getGridLineColor() {
        return gridLineColor;
    }

    /**
     * <pre>
     * 获取值对应X点坐标
     * </pre>
     * 
     * @param value
     * @return
     */
    private int getXPoint(long value) {
        Long minXCoords = dataModel.getMinXCoords();
        long v = value - minXCoords;
        int x = (int) (v * xRatio);
        return x + YTextWidth;
    }

    /**
     * <pre>
     * 获取值对应Y点坐标
     * </pre>
     * 
     * @param value
     * @return
     */
    private int getYPoint(long value) {
        Long minYCoords = dataModel.getMinYCoords();
        long v = value - minYCoords;
        int y = (int) (v * yRatio);
        int graphHeight = getGraphHeight();
        return graphHeight - y + titleHeight;
    }

    public boolean isDrawEmptyGraph() {
        return drawEmptyGraph;
    }

    @Override
    public void monitorPanelChanged(MonitorPanelModelEvent e) {
        repaint();
    }

    @Override
    public void paint(Graphics g) {
        super.paint(g);

        drawImage();
        // 用图片
        if (bufferImage != null) {
            g.drawImage(bufferImage, 0, 0, this);
        }
    }

    public void setCurrentTextWidth(int currentTextWidth) {
        this.currentTextWidth = currentTextWidth;
    }

    public void setDisplayLastValue(boolean display) {
        displayLastValue = display;
        if (display) {
            currentTextWidth = defaultCurrentTextWidth;
        } else {
            currentTextWidth = 0;
        }
    }

    public void setDrawEmptyGraph(boolean drawEmptyGraph) {
        this.drawEmptyGraph = drawEmptyGraph;
    }

    public void setFontColor(Color fontColor) {
        this.fontColor = fontColor;
    }

    public void setGraphBackgroundColor(Color graphBackgroundColor) {
        this.graphBackgroundColor = graphBackgroundColor;
    }

    public void setGraphLineColor(Color graphLineColor) {
        this.graphLineColor = graphLineColor;
    }

    public void setGraphTitle(String title) {
        this.graphTitle = title;
    }

    public void setGridLineColor(Color gridLineColor) {
        this.gridLineColor = gridLineColor;
    }

    public void setModel(MonitorPanelModel model) {
        if (model == null) {
            throw new IllegalArgumentException("不能为空");
        }
        if (this.dataModel != model) {
            MonitorPanelModel old = this.dataModel;
            if (old != null) {
                old.removeMonitorPanelModelListener(this);
            }
            this.dataModel = model;
            model.addMonitorPanelModelListener(this);
            repaint();
        }
    }

}

```

### 数据模型 MonitorPanelModel
```
package com.wwh.test.swing.monitor;

import java.io.Serializable;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import javax.swing.event.EventListenerList;

// 应该抽象出接口
public class MonitorPanelModel implements Serializable {

    public enum XAxisMode {
        AUTO, FIXED, KEEP_MOVE
    }

    public enum YAxisMode {
        AUTO, FIXED
    }

    /**
     * 默认的最大坐标点的数量1000
     */
    public static final int DEFAULT_MAX_COORDINATE_NUMBER = 1000;
    private static final long serialVersionUID = 1L;
    /**
     * 坐标集合
     */
    private List<Coordinate> coordinatelist;
    /**
     * 使X轴呈现一个向左边移动的效果<br>
     * 需要指定一个时间段
     */
    private Long keepMoveTimeInterval;
    private EventListenerList listenerList = new EventListenerList();
    /**
     * 最大的坐标点个数
     */
    private int maxCoordinateCount = DEFAULT_MAX_COORDINATE_NUMBER;
    /**
     * 最大X坐标
     */
    private Long maxXCoords;
    /**
     * X轴上的最大值
     */
    private Long maxXValue;
    /**
     * 最大Y坐标
     */
    private Long maxYCoords;
    /**
     * Y轴上的最大值
     */
    private Long maxYValue;
    /**
     * 最小X坐标
     */
    private Long minXCoords;
    /**
     * X轴上的最小值
     */
    private Long minXValue;
    /**
     * 最小Y坐标
     */
    private Long minYCoords;

    /**
     * Y轴上的最小值
     */
    private Long minYValue;

    /**
     * 记录一个开始的时间
     */
    private Long startTime;

    /**
     * X 轴间距
     */
    private Long xAxisInterval;

    private XAxisMode xAxisMode = XAxisMode.AUTO;

    /**
     * Y轴间距
     */
    private Long yAxisInterval;

    private YAxisMode yAxisMode = YAxisMode.AUTO;

    public MonitorPanelModel() {
        coordinatelist = new ArrayList<Coordinate>();
    }

    public void addCoordinate(Coordinate coordinate) {
        coordinatelist.add(coordinate);
        if (coordinatelist.size() > maxCoordinateCount) {
            coordinatelist.remove(0);
            fireMonitorPanelDataChanged();
        } else {
            fireMonitorPanelDataAdd(coordinate);
        }
    }

    public void addMonitorPanelModelListener(MonitorPanelModelListener l) {
        listenerList.add(MonitorPanelModelListener.class, l);
    }

    /**
     * 根据间隔计算Y轴上的线条数量
     * 
     * @param interval
     * @return
     */
    private int calcLineCountByInterval(long interval) {
        // 计算Y轴的最大值
        long tmp = maxYValue / interval;
        maxYCoords = tmp * interval;
        if (maxYCoords < maxYValue) {
            maxYCoords += interval;
        }

        tmp = minYValue / interval;
        minYCoords = tmp * interval;
        if (minYCoords > minYValue) {
            minYCoords -= interval;
        }

        long line = (maxYCoords - minYCoords) / interval;

        yAxisInterval = interval;

        return (int) line;
    }

    public void clearCoordinate() {
        coordinatelist.clear();
        fireMonitorPanelDataChanged();
    }

    private void fireMonitorPanelChanged(MonitorPanelModelEvent e) {
        Object[] listeners = listenerList.getListenerList();
        for (int i = listeners.length - 2; i >= 0; i -= 2) {
            if (listeners[i] == MonitorPanelModelListener.class) {
                ((MonitorPanelModelListener) listeners[i + 1]).monitorPanelChanged(e);
            }
        }
    }

    private void fireMonitorPanelDataAdd(Coordinate coordinate) {
        refreshValue(coordinate);
        fireMonitorPanelChanged(new MonitorPanelModelEvent(this));
    }

    private void fireMonitorPanelDataChanged() {
        // 刷新数据
        refreshAllValue();
        fireMonitorPanelChanged(new MonitorPanelModelEvent(this));
    }

    private void fireMonitorPanelPropertyChanged(String property, Object oldValue, Object newValue) {
        // 属性先不管
        fireMonitorPanelChanged(new MonitorPanelModelEvent(this));
    }

    private long[] getAutoXAxisLines(int bestXLineCount) {
        if (maxXValue == null || minXValue == null) {
            return null;
        }
        long differ = maxXValue - minXValue;
        if (differ == 0) {
            return null;
        }
        minXCoords = minXValue;
        maxXCoords = maxXValue;
        xAxisInterval = differ / bestXLineCount;
        if (xAxisInterval < 1) {
            return null;
        }
        List<Long> l = new ArrayList<>();
        long tmp = maxXCoords;
        while (tmp > minXCoords) {
            l.add(tmp);
            tmp -= xAxisInterval;
        }
        long[] ret = new long[l.size()];
        for (int i = 0; i < ret.length; i++) {
            ret[i] = l.get(i);
        }
        return ret;
    }

    private long[] getAutoYAxisLines(int bestYLineCount) {
        if (maxYValue == null || minYValue == null) {
            return null;
        }
        long differValue = maxYValue - minYValue;
        if (differValue == 0) {
            // 此时可能是一条横线
            yAxisInterval = 1L;
            long absMaxYValue = Math.abs(maxYValue);
            if (absMaxYValue > 10) {
                double log = Math.log10(absMaxYValue / 10);
                // 四舍五入取整
                log = Math.round(log);
                // 间距
                yAxisInterval = (long) Math.pow(10, log);
            }
            maxYCoords = ((maxYValue / yAxisInterval) + 1) * yAxisInterval;
            minYCoords = ((maxYValue / yAxisInterval) - 1) * yAxisInterval;
            long middleValue = (maxYValue / yAxisInterval) * yAxisInterval;
            return new long[] { minYCoords, middleValue, maxYCoords };
        }

        long interval = 1;
        if (differValue > 10) {
            double log = Math.log10(differValue / 10);
            // 四舍五入取整
            log = Math.round(log);
            // 间距
            interval = (long) Math.pow(10, log);
        }
        readjustYAxis(interval, bestYLineCount);

        // 返回Y轴上的坐标线
        return getFixedYAxisLines();
    }

    public List<Coordinate> getCoordinatelist() {
        return Collections.unmodifiableList(coordinatelist);
    }

    public int getCoordinateSize() {
        return coordinatelist.size();
    }

    private long[] getFixedXAxisLines() {
        if (maxXCoords == null || minXCoords == null || xAxisInterval == null) {
            throw new IllegalArgumentException("固定模式下，最大值、最小值、间距都不能为空");
        }
        List<Long> l = new ArrayList<>();
        long tmp = minXCoords;
        while (tmp <= maxXCoords) {
            l.add(tmp);
            tmp += xAxisInterval;
        }
        long[] ret = new long[l.size()];
        for (int i = 0; i < ret.length; i++) {
            ret[i] = l.get(i);
        }
        return ret;
    }

    private long[] getFixedYAxisLines() {
        if (maxYCoords == null || minYCoords == null || yAxisInterval == null) {
            throw new IllegalArgumentException("固定模式下，最大值、最小值、间距都不能为空");
        }
        List<Long> l = new ArrayList<>();
        long tmp = minYCoords;
        while (tmp <= maxYCoords) {
            l.add(tmp);
            tmp += yAxisInterval;
        }
        long[] ret = new long[l.size()];
        for (int i = 0; i < ret.length; i++) {
            ret[i] = l.get(i);
        }
        return ret;
    }

    public Long getKeepMoveTimeInterval() {
        return keepMoveTimeInterval;
    }

    private long[] getKeepMoveXAxisLines(int bestXLineCount) {
        if (keepMoveTimeInterval == null || keepMoveTimeInterval < 1) {
            throw new IllegalArgumentException("移动模式下，固定的时间段不能为空或小于1");
        }
        if (startTime == null) {
            startTime = System.currentTimeMillis();
        }
        maxXCoords = System.currentTimeMillis();
        minXCoords = maxXCoords - keepMoveTimeInterval;

        xAxisInterval = keepMoveTimeInterval / bestXLineCount;
        if (xAxisInterval < 1) {
            return null;
        }
        List<Long> l = new ArrayList<>();
        long tmp = startTime;
        while (tmp <= maxXCoords) {
            if (tmp >= minXCoords) {
                l.add(tmp);
            }
            tmp += xAxisInterval;
        }
        long[] ret = new long[l.size()];
        for (int i = 0; i < ret.length; i++) {
            ret[i] = l.get(i);
        }
        return ret;
    }

    public int getMaxCoordinateCount() {
        return maxCoordinateCount;
    }

    public Long getMaxXCoords() {
        return maxXCoords;
    }

    public Long getMaxXValue() {
        return maxXValue;
    }

    public Long getMaxYCoords() {
        return maxYCoords;
    }

    public Long getMaxYValue() {
        return maxYValue;
    }

    public Long getMinXCoords() {
        return minXCoords;
    }

    public Long getMinXValue() {
        return minXValue;
    }

    public Long getMinYCoords() {
        return minYCoords;
    }

    public Long getMinYValue() {
        return minYValue;
    }

    public MonitorPanelModelListener[] getMonitorPanelModelListeners() {
        return listenerList.getListeners(MonitorPanelModelListener.class);
    }

    public long getXAxisFirstValue() {
        if (coordinatelist.isEmpty()) {
            return 0;
        }
        return coordinatelist.get(0).getxValue();
    }

    public Long getxAxisInterval() {
        return xAxisInterval;
    }

    public long getXAxisLastValue() {
        if (coordinatelist.isEmpty()) {
            return 0;
        }
        return coordinatelist.get(coordinatelist.size() - 1).getxValue();
    }

    /**
     * X 轴上要画的竖线
     * 
     * @param bestXLineCount 最佳的竖线条数
     * @return
     */
    public long[] getXAxisLines(int bestXLineCount) {

        switch (xAxisMode) {
        case AUTO:
            return getAutoXAxisLines(bestXLineCount);
        case FIXED:
            return getFixedXAxisLines();
        case KEEP_MOVE:
            return getKeepMoveXAxisLines(bestXLineCount);
        default:
            return null;
        }
    }

    public XAxisMode getxAxisMode() {
        return xAxisMode;
    }

    public Long getyAxisInterval() {
        return yAxisInterval;
    }

    /**
     * Y 轴上要画的横线
     * 
     * @param bestYLineCount 最佳的横线条数
     * @return
     */
    public long[] getYAxisLines(int bestYLineCount) {
        switch (yAxisMode) {
        case AUTO:
            return getAutoYAxisLines(bestYLineCount);
        case FIXED:
            return getFixedYAxisLines();
        default:
            return null;
        }
    }

    public YAxisMode getyAxisMode() {
        return yAxisMode;
    }

    /**
     * 重新计算Y轴
     * 
     * @param interval
     */
    private void readjustYAxis(long interval, int bestYLineCount) {
        // 先根据目前的间距计算一次Y轴的行数
        int lineCount = calcLineCountByInterval(interval);

        // 再尝试调整为最佳的
        long newInterval = 0;
        if (bestYLineCount > lineCount) {
            // 减小间距
            float f = (float) bestYLineCount / lineCount;
            int p = Math.round(f);
            if (p == 1) {
                return;
            } else {
                for (int i = p; i > 1; i--) {
                    if (interval % i == 0) {
                        newInterval = interval / i;
                        break;
                    }
                }
            }
        } else {
            // 增加间距
            float f = (float) lineCount / bestYLineCount;
            int p = Math.round(f);
            if (p == 1) {
                return;
            } else {
                newInterval = interval * p;
            }
        }

        if (newInterval != 0) {
            // 根据调整后的间距再算一次
            lineCount = calcLineCountByInterval(newInterval);
        }
    }

    private void refreshAllValue() {
        minXValue = null;
        maxXValue = null;
        minYValue = null;
        maxYValue = null;
        for (Coordinate coordinate : coordinatelist) {
            refreshValue(coordinate);
        }
    }

    private void refreshValue(Coordinate coordinate) {
        long y = coordinate.getyValue();
        long x = coordinate.getxValue();
        if (maxYValue == null || y > maxYValue) {
            maxYValue = y;
        }
        if (minYValue == null || y < minYValue) {
            minYValue = y;
        }
        if (maxXValue == null || x > maxXValue) {
            maxXValue = x;
        }
        if (minXValue == null || x < minXValue) {
            minXValue = x;
        }
    }

    public void removeMonitorPanelModelListener(MonitorPanelModelListener l) {
        listenerList.remove(MonitorPanelModelListener.class, l);
    }

    public void setCoordinatelist(List<Coordinate> coordinatelist) {
        if (coordinatelist == null) {
            throw new IllegalArgumentException("不能为空");
        }
        this.coordinatelist = coordinatelist;
        fireMonitorPanelDataChanged();
    }

    public void setKeepMoveTimeInterval(Long keepMoveTimeInterval) {
        fireMonitorPanelPropertyChanged(null, null, null);
        this.keepMoveTimeInterval = keepMoveTimeInterval;
    }

    public void setMaxCoordinateCount(int maxCoordinateCount) {
        fireMonitorPanelPropertyChanged(null, null, null);
        this.maxCoordinateCount = maxCoordinateCount;
    }

    public void setMaxXCoords(Long maxXCoords) {
        fireMonitorPanelPropertyChanged(null, null, null);
        this.maxXCoords = maxXCoords;
    }

    public void setMaxYCoords(Long maxYCoords) {
        fireMonitorPanelPropertyChanged(null, null, null);
        this.maxYCoords = maxYCoords;
    }

    public void setMinXCoords(Long minXCoords) {
        fireMonitorPanelPropertyChanged(null, null, null);
        this.minXCoords = minXCoords;
    }

    public void setMinYCoords(Long minYCoords) {
        fireMonitorPanelPropertyChanged(null, null, null);
        this.minYCoords = minYCoords;
    }

    public void setxAxisInterval(Long xAxisInterval) {
        fireMonitorPanelPropertyChanged(null, null, null);
        this.xAxisInterval = xAxisInterval;
    }

    public void setxAxisMode(XAxisMode xAxisMode) {
        fireMonitorPanelPropertyChanged(null, null, null);
        this.xAxisMode = xAxisMode;
    }

    public void setyAxisInterval(Long yAxisInterval) {
        fireMonitorPanelPropertyChanged(null, null, null);
        this.yAxisInterval = yAxisInterval;
    }

    public void setyAxisMode(YAxisMode yAxisMode) {
        fireMonitorPanelPropertyChanged(null, null, null);
        this.yAxisMode = yAxisMode;
    }

}

```
### 坐标 Coordinate
```
package com.wwh.test.swing.monitor;

/**
 * <pre>
 * xy坐标
 * X轴 为 具体的值
 * Y轴 为 时间
 * </pre>
 * 
 * @author wwh
 */
public class Coordinate {

    private long xValue;
    private long yValue;

    public Coordinate(long x, long y) {
        this.xValue = x;
        this.yValue = y;
    }

    public long getxValue() {
        return xValue;
    }

    public void setxValue(long xValue) {
        this.xValue = xValue;
    }

    public long getyValue() {
        return yValue;
    }

    public void setyValue(long yValue) {
        this.yValue = yValue;
    }

}

```

### 事件 MonitorPanelModelEvent
```
package com.wwh.test.swing.monitor;

public class MonitorPanelModelEvent extends java.util.EventObject {
    private static final long serialVersionUID = 1L;

    public MonitorPanelModelEvent(Object source) {
        super(source);
    }

}

```
### 监听器接口 MonitorPanelModelListener
```
package com.wwh.test.swing.monitor;

public interface MonitorPanelModelListener extends java.util.EventListener {

    public void monitorPanelChanged(MonitorPanelModelEvent e);
}

```

### 测试 MonitorPanelTest
```
package com.wwh.test.swing.monitor;

import java.awt.BorderLayout;
import java.awt.Color;
import java.awt.EventQueue;
import java.awt.GridLayout;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.util.Random;

import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.border.EmptyBorder;

import com.wwh.test.swing.monitor.MonitorPanelModel.XAxisMode;
import com.wwh.test.swing.monitor.MonitorPanelModel.YAxisMode;

public class MonitorPanelTest extends JFrame {

    private static final long serialVersionUID = 1L;

    /**
     * Launch the application.
     */
    public static void main(String[] args) {
        EventQueue.invokeLater(new Runnable() {
            public void run() {
                try {
                    MonitorPanelTest frame = new MonitorPanelTest();
                    frame.setVisible(true);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });
    }

    private JButton btn_clear1;
    private JButton btn_clear2;
    private JButton btn_clear3;
    private JButton btn_clear4;

    private JButton btn_start2;
    private JButton btn_start3;
    private JButton btn_start4;

    private JButton btn_stop1;
    private JButton btn_stop2;
    private JButton btn_stop3;
    private JButton btn_stop4;

    private JPanel contentPane;
    private MonitorPanel mPanel1;
    private MonitorPanel mPanel2;
    private MonitorPanel mPanel3;
    private MonitorPanel mPanel4;

    private MonitorPanelModel mpModel1;
    private MonitorPanelModel mpModel2;
    private MonitorPanelModel mpModel3;
    private MonitorPanelModel mpModel4;
    private JPanel panel_1;
    private JPanel panel_2;
    private JPanel panel_2_btn;
    private JPanel panel_3;
    private JPanel panel_3_btn;
    private JPanel panel_4;
    private JPanel panel_4_btn;
    private boolean runFlag1 = false;
    private boolean runFlag2 = false;
    private boolean runFlag3 = false;
    private boolean runFlag4 = false;

    /**
     * Create the frame.
     */
    public MonitorPanelTest() {
        setTitle("监控面板测试");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setBounds(100, 100, 1183, 681);
        contentPane = new JPanel();
        contentPane.setBorder(new EmptyBorder(5, 5, 5, 5));
        setContentPane(contentPane);
        contentPane.setLayout(new GridLayout(2, 2, 0, 0));

        createMonitorPanel1();

        createMonitorPanel2();

        createMonitorPanel3();

        createMonitorPanel4();

    }

    private void createMonitorPanel1() {
        panel_1 = new JPanel();
        contentPane.add(panel_1);
        panel_1.setLayout(new BorderLayout(0, 0));
        mpModel1 = new MonitorPanelModel();
        mPanel1 = new MonitorPanel(mpModel1);
        panel_1.add(mPanel1);

        mpModel1.setMaxCoordinateCount(200);

        mPanel1.setGraphTitle("0到10000之间的随机数");

        JPanel panel_1_btn = new JPanel();
        panel_1.add(panel_1_btn, BorderLayout.SOUTH);

        JButton btn_start1 = new JButton("启  动");
        btn_start1.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                start1();
            }
        });
        panel_1_btn.add(btn_start1);

        btn_stop1 = new JButton("停  止");
        btn_stop1.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                runFlag1 = false;
            }
        });
        panel_1_btn.add(btn_stop1);

        btn_clear1 = new JButton("清  空");
        btn_clear1.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                mpModel1.clearCoordinate();
            }
        });
        panel_1_btn.add(btn_clear1);
    }

    private void createMonitorPanel2() {
        panel_2 = new JPanel();
        contentPane.add(panel_2);
        panel_2.setLayout(new BorderLayout(0, 0));

        mpModel2 = new MonitorPanelModel();
        mpModel2.setxAxisMode(XAxisMode.KEEP_MOVE);
        mpModel2.setKeepMoveTimeInterval(10 * 60 * 1000L);

        mpModel2.setMaxCoordinateCount(600);

        mPanel2 = new MonitorPanel(mpModel2);

        mPanel2.setGraphTitle("内存图测试");
        mPanel2.setGraphBackgroundColor(new Color(38, 38, 38));
        mPanel2.setGridLineColor(new Color(0, 225, 34));
        mPanel2.setGraphLineColor(Color.YELLOW);

        panel_2.add(mPanel2);

        panel_2_btn = new JPanel();
        panel_2.add(panel_2_btn, BorderLayout.SOUTH);

        btn_start2 = new JButton("启  动");
        btn_start2.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                start2();
            }
        });
        panel_2_btn.add(btn_start2);

        btn_stop2 = new JButton("停  止");
        btn_stop2.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                runFlag2 = false;
            }
        });
        panel_2_btn.add(btn_stop2);

        btn_clear2 = new JButton("清  空");
        btn_clear2.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                mpModel2.clearCoordinate();
            }
        });
        panel_2_btn.add(btn_clear2);
    }

    private void createMonitorPanel3() {
        panel_3 = new JPanel();
        contentPane.add(panel_3);
        panel_3.setLayout(new BorderLayout(0, 0));

        mpModel3 = new MonitorPanelModel();
        mpModel3.setxAxisMode(XAxisMode.KEEP_MOVE);
        mpModel3.setKeepMoveTimeInterval(10 * 60 * 1000L);
        mpModel3.setMaxCoordinateCount(1200);

        mPanel3 = new MonitorPanel(mpModel3);
        mPanel3.setGraphTitle("递增递减");

        panel_3.add(mPanel3, BorderLayout.CENTER);

        panel_3_btn = new JPanel();
        panel_3.add(panel_3_btn, BorderLayout.SOUTH);

        btn_start3 = new JButton("启  动");
        btn_start3.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                start3();
            }
        });
        panel_3_btn.add(btn_start3);

        btn_stop3 = new JButton("停  止");
        btn_stop3.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                runFlag3 = false;
            }
        });
        panel_3_btn.add(btn_stop3);

        btn_clear3 = new JButton("清  空");
        btn_clear3.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                mpModel3.clearCoordinate();
            }
        });
        panel_3_btn.add(btn_clear3);

    }

    private void createMonitorPanel4() {

        panel_4 = new JPanel();
        contentPane.add(panel_4);
        panel_4.setLayout(new BorderLayout(0, 0));

        mpModel4 = new MonitorPanelModel();
        mpModel4.setxAxisMode(XAxisMode.KEEP_MOVE);
        mpModel4.setKeepMoveTimeInterval(360 * 1000L);

        mpModel4.setMaxCoordinateCount(720);

        mpModel4.setyAxisMode(YAxisMode.FIXED);
        mpModel4.setMaxYCoords(100L);
        mpModel4.setMinYCoords(-100L);
        mpModel4.setyAxisInterval(20L);

        mPanel4 = new MonitorPanel(mpModel4);
        panel_4.add(mPanel4, BorderLayout.CENTER);

        mPanel4.setGraphTitle("正弦函数");
        mPanel4.setGraphBackgroundColor(new Color(38, 38, 38));
        mPanel4.setGridLineColor(new Color(0, 225, 34));
        mPanel4.setGraphLineColor(Color.YELLOW);

        panel_4_btn = new JPanel();
        panel_4.add(panel_4_btn, BorderLayout.SOUTH);

        btn_start4 = new JButton("启  动");
        btn_start4.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                start4();
            }
        });
        panel_4_btn.add(btn_start4);

        btn_stop4 = new JButton("停  止");
        btn_stop4.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                runFlag4 = false;
            }
        });
        panel_4_btn.add(btn_stop4);

        btn_clear4 = new JButton("清  空");
        btn_clear4.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                mpModel4.clearCoordinate();
            }
        });
        panel_4_btn.add(btn_clear4);

    }

    private void start1() {
        runFlag1 = true;
        Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                int rint;
                Random r = new Random();
                while (runFlag1) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    rint = r.nextInt(10000);
                    mpModel1.addCoordinate(new Coordinate(System.currentTimeMillis(), rint + 1));
                }
            }
        });
        t.start();
    }

    private void start2() {
        if (runFlag2) {
            return;
        }
        runFlag2 = true;
        Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                Runtime rt = Runtime.getRuntime();

                while (runFlag2) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    mpModel2.addCoordinate(new Coordinate(System.currentTimeMillis(), rt.freeMemory() / 1024 / 1024));
                }
            }
        });
        t.start();
    }

    private void start3() {

        if (runFlag3) {
            return;
        }
        runFlag3 = true;
        Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                boolean b = true;
                int i = 0;
                while (runFlag3) {
                    if (b) {
                        if (i < 100) {
                            i++;
                        } else {
                            b = !b;
                        }
                    } else {
                        if (i > 0) {
                            i--;
                        } else {
                            b = !b;
                        }
                    }
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    mpModel3.addCoordinate(new Coordinate(System.currentTimeMillis(), i));
                }
            }
        });
        t.start();

    }

    private void start4() {
        if (runFlag4) {
            return;
        }
        runFlag4 = true;
        Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                int i = 0;
                double d;
                long l;
                while (runFlag4) {
                    if (i == 360) {
                        i = 0;
                    }
                    d = Math.sin(Math.toRadians(i));
                    i++;
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    l = (long) (d * 100);
                    mpModel4.addCoordinate(new Coordinate(System.currentTimeMillis(), l));
                }
            }
        });
        t.start();
    }
}

```

