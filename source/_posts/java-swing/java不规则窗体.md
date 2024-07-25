---
title: java不规则窗体
date: 2020-02-07 23:41
tags: 
  - java-swing
categories:
  - [java-swing]
---

用背景透明的png图片

![不规则窗体](https://upload-images.jianshu.io/upload_images/2043910-0284c9db586fc2de.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 不规则窗体
```
package com.wwh.excel.tools.swing;

import java.awt.Color;
import java.awt.Component;
import java.awt.Graphics;
import java.awt.Image;
import java.awt.Point;
import java.awt.Toolkit;
import java.awt.datatransfer.DataFlavor;
import java.awt.dnd.DnDConstants;
import java.awt.dnd.DropTarget;
import java.awt.dnd.DropTargetAdapter;
import java.awt.dnd.DropTargetDropEvent;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;
import java.awt.event.MouseMotionAdapter;
import java.io.File;
import java.util.List;

import javax.swing.ImageIcon;
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JOptionPane;

import com.wwh.excel.tools.worker.Csv2Xls;

public class ImageFrame extends JFrame {

    private static final long serialVersionUID = 1L;
    private ImageIcon icon;
    private Point origin = new Point();; // 用于移动窗体

    // 小弹出框
    private PopupDialog popupDialog;

    public ImageFrame(ImageIcon icon2) {
        this.icon = icon2;
        JLabel imageLabel = new JLabel() {
            private static final long serialVersionUID = 1L;

            @Override
            public void paint(Graphics g) {
                super.paint(g);
                icon.paintIcon(this, g, 0, 0);
            }
        };
        this.add(imageLabel);

        this.setUndecorated(true); // 关键语句1 不启用窗体装饰

        this.setSize(icon.getIconWidth(), icon.getIconHeight()); // 设置窗口大小

        // 这个需要包的支持
        // AWTUtilities.setWindowOpaque(this, false);

        // JDK 1.7 版本以上，使用 this.setBackground(new Color(0, 0, 0, 0));
        this.setBackground(new Color(0, 0, 0, 0)); // 关键句2

        // 设置透明度
        this.setOpacity(0.9f);

        this.setLocationRelativeTo(null); // 设置窗口居中
        // setVisible(true);

        this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        // 鼠标事件监听
        // 由于取消了默认的窗体结构，所以我们要手动设置一下移动窗体的方法
        this.addMouseListener(new MouseAdapter() {
            public void mousePressed(MouseEvent e) {
                origin.x = e.getX();
                origin.y = e.getY();
            }

            // 窗体上单击鼠标右键关闭程序
            public void mouseClicked(MouseEvent e) {

                if (e.getButton() == MouseEvent.BUTTON3) {
                    System.exit(0);
                } else {
                    if (popupDialog != null) {
                        popupDialog.showDelayHide();
                    }
                }
            }

        });

        this.addMouseMotionListener(new MouseMotionAdapter() {
            public void mouseDragged(MouseEvent e) {
                Point p = getLocation();
                setLocation(p.x + e.getX() - origin.x, p.y + e.getY() - origin.y);
            }
        });
        drag(imageLabel);

        // 设置窗口图标
        Image img2 = Toolkit.getDefaultToolkit().getImage(ImageFrame.class.getResource("/icon.png"));
        setIconImage(img2);
    }

    // 定义的拖拽方法
    public void drag(Component c) {
        // c 表示要接受拖拽的控件
        new DropTarget(c, DnDConstants.ACTION_COPY_OR_MOVE, new DropTargetAdapter() {
            @Override
            public void drop(DropTargetDropEvent dtde)// 重写适配器的drop方法
            {
                try {
                    if (dtde.isDataFlavorSupported(DataFlavor.javaFileListFlavor))// 如果拖入的文件格式受支持
                    {
                        dtde.acceptDrop(DnDConstants.ACTION_COPY_OR_MOVE);// 接收拖拽来的数据

                        @SuppressWarnings("unchecked")
                        List<File> list = (List<File>) (dtde.getTransferable()
                                .getTransferData(DataFlavor.javaFileListFlavor));

                        if (list.size() > 1) {
                            JOptionPane.showMessageDialog(null, "一次拖一个文件，靓仔");
                            return;
                        }

                        Csv2Xls.convert(list.get(0));

                        popupDialog.showDelayHide();

                        dtde.dropComplete(true);// 指示拖拽操作已完成

                    } else {
                        dtde.rejectDrop();// 否则拒绝拖拽来的数据
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                    JOptionPane.showMessageDialog(null, "出错了，靓仔\n" + e.getMessage());
                }
            }
        });
    }

    public PopupDialog getPopupDialog() {
        return popupDialog;
    }

    public void setPopupDialog(PopupDialog popupDialog) {
        this.popupDialog = popupDialog;
    }

}

```

### 不规则弹出框
```
package com.wwh.excel.tools.swing;

import java.awt.Color;
import java.awt.Frame;
import java.awt.Graphics;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;

import javax.swing.ImageIcon;
import javax.swing.JDialog;
import javax.swing.JLabel;
import javax.swing.Timer;

public class PopupDialog extends JDialog {

    private static final long serialVersionUID = 1L;

    private ImageIcon icon;

    private Frame owner;

    private Timer timer;

    private void initTimer(int delayTime) {
        // 定时消失
        ActionListener taskPerformer = new ActionListener() {
            public void actionPerformed(ActionEvent evt) {
                setVisible(false);
            }
        };
        timer = new Timer(delayTime, taskPerformer);
        timer.setRepeats(false);
    }

    /**
     * Create the dialog.
     */
    public PopupDialog(Frame owner, ImageIcon icon2, int delayTime) {
        super(owner);
        this.icon = icon2;
        this.owner = owner;

        initTimer(delayTime);

        JLabel imageLabel = new JLabel() {
            private static final long serialVersionUID = 1L;

            @Override
            public void paint(Graphics g) {
                super.paint(g);
                icon.paintIcon(this, g, 0, 0);
            }
        };

        this.add(imageLabel);

        this.setUndecorated(true); // 关键语句1 不启用窗体装饰

        this.setSize(icon.getIconWidth(), icon.getIconHeight()); // 设置窗口大小

        // 这个需要包的支持
        // AWTUtilities.setWindowOpaque(this, false);

        // JDK 1.7 版本以上，使用 this.setBackground(new Color(0, 0, 0, 0));
        this.setBackground(new Color(0, 0, 0, 0)); // 关键句2

        this.setLocationRelativeTo(owner);

        this.setDefaultCloseOperation(HIDE_ON_CLOSE);

        this.addMouseListener(new MouseAdapter() {
            @Override
            public void mouseClicked(MouseEvent e) {

                PopupDialog.this.setVisible(false);

            }
        });
    }

    /**
     * 先显示再延期隐藏
     */
    public void showDelayHide() {
        this.setLocationRelativeTo(owner);
        setVisible(true);
        timer.start();
    }

    @Override
    public void setVisible(boolean b) {
        super.setVisible(b);
        if (!b) {
            timer.stop();
        }
    }

    /**
     * Launch the application.
     */
    public static void main(String[] args) {
        try {
            ImageIcon icon = new ImageIcon(PopupDialog.class.getResource("/images/success.png"));
            PopupDialog dialog = new PopupDialog(null, icon, 10000);
            dialog.setDefaultCloseOperation(JDialog.DISPOSE_ON_CLOSE);
            // dialog.setVisible(true);
            dialog.showDelayHide();

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}

```
