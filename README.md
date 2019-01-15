# AndroidScreenFit
Android ScreenFit

### 百分比的引入
[Hongyang Android 屏幕适配方案](http://blog.csdn.net/lmj623565791/article/details/45460089)

[jar包下载]()

```
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.PrintWriter;

/**
 * ClassName: GenerateValueFiles <br/>
 * Description: 按比例生成不同尺寸 <br/>
 * 1.修改支持尺寸 SUPPORT_DIMESION
 * 2.修改main切图比例
 */
public class GenerateValueFiles {

    private int baseW;
    private int baseH;

    private String dirStr = "./res1";

    private final static String WTemplate = "<dimen name=\"x{0}\">{1}px</dimen>\n";
    private final static String HTemplate = "<dimen name=\"y{0}\">{1}px</dimen>\n";
    private final static String WTemplateDp = "<dimen name=\"x{0}\">{1}dp</dimen>\n";
    private final static String HTemplateDp = "<dimen name=\"y{0}\">{1}dp</dimen>\n";

    /**
     * {0}-HEIGHT
     */
    private final static String VALUE_TEMPLATE = "values-{0}x{1}";
    private final static String VALUE_TEMPLATE_STAND = "values";
    private final static String VALUE_TEMPLATE_LAND = "values-land-{0}x{1}";

    private static final String SUPPORT_DIMESION = "320,480;480,800;480,854;540,960;600,1024;720," +
            "1184;720,1196;720,1280;768,1024;768,1280;800,1280;1080,1812;1080,1920;1200,1920;1440,2560;";

    private String supportStr = SUPPORT_DIMESION;

    public GenerateValueFiles(int baseX, int baseY, String supportStr) {
        this.baseW = baseX;
        this.baseH = baseY;

        if (!this.supportStr.contains(baseX + "," + baseY)) {
            this.supportStr += baseX + "," + baseY + ";";
        }

        this.supportStr += validateInput(supportStr);

        System.out.println(supportStr);

        File dir = new File(dirStr);
        if (!dir.exists()) {
            dir.mkdir();

        }
        System.out.println(dir.getAbsoluteFile());

    }

    /**
     * @param supportStr w,h_...w,h;
     * @return
     */
    private String validateInput(String supportStr) {
        StringBuffer sb = new StringBuffer();
        String[] vals = supportStr.split("_");
        int w = -1;
        int h = -1;
        String[] wh;
        for (String val : vals) {
            try {
                if (val == null || val.trim().length() == 0)
                    continue;

                wh = val.split(",");
                w = Integer.parseInt(wh[0]);
                h = Integer.parseInt(wh[1]);
            } catch (Exception e) {
                System.out.println("skip invalidate params : w,h = " + val);
                continue;
            }
            sb.append(w + "," + h + ";");
        }

        return sb.toString();
    }

    public void generate() {
        String[] vals = supportStr.split(";");
        int max = 0;
        if (baseW >= baseH) {
            max = baseW;
        } else {
            max = baseH;
        }
        for (String val : vals) {
            String[] wh = val.split(",");
            generateXmlFile(Integer.parseInt(wh[0]), Integer.parseInt(wh[1]), max);
            generateXmlLandFile(Integer.parseInt(wh[0]), Integer.parseInt(wh[1]), max);
        }
        generateXmlStandFile(max, 2);
    }

    private void generateXmlStandFile(int max, float scale) {

        StringBuffer sbForWidth = new StringBuffer();
        sbForWidth.append("<?xml version=\"1.0\" encoding=\"utf-8\"?>\n");
        sbForWidth.append("<resources>");
        System.out.println("max : " + max + "," + scale + "," + scale);
        for (int i = 1; i <= max; i++) {
            sbForWidth.append(WTemplateDp.replace("{0}", i + "").replace("{1}",
                    changedp(i, scale) + ""));
        }
//		sbForWidth.append(WTemplate.replace("{0}", baseW + "").replace("{1}",
//				w + ""));
        sbForWidth.append("</resources>");

        StringBuffer sbForHeight = new StringBuffer();
        sbForHeight.append("<?xml version=\"1.0\" encoding=\"utf-8\"?>\n");
        sbForHeight.append("<resources>");

        for (int i = 1; i <= max; i++) {
            sbForHeight.append(HTemplateDp.replace("{0}", i + "").replace("{1}",
                    changedp(i, scale) + ""));
        }
//		sbForHeight.append(HTemplate.replace("{0}", baseH + "").replace("{1}",
//				h + ""));
        sbForHeight.append("</resources>");

        File fileDir = new File(dirStr + File.separator
                + VALUE_TEMPLATE_STAND);
        fileDir.mkdir();

        File layxFile = new File(fileDir.getAbsolutePath(), "lay_x.xml");
        File layyFile = new File(fileDir.getAbsolutePath(), "lay_y.xml");
        try {
            PrintWriter pw = new PrintWriter(new FileOutputStream(layxFile));
            pw.print(sbForWidth.toString());
            pw.close();
            pw = new PrintWriter(new FileOutputStream(layyFile));
            pw.print(sbForHeight.toString());
            pw.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }

    private void generateXmlLandFile(int w, int h, int max) {

        StringBuffer sbForWidth = new StringBuffer();
        sbForWidth.append("<?xml version=\"1.0\" encoding=\"utf-8\"?>\n");
        sbForWidth.append("<resources>");
        float cellw = w * 1.0f / baseW;

        System.out.println("width : " + w + "," + baseW + "," + cellw);
        for (int i = 1; i <= max; i++) {
            sbForWidth.append(HTemplate.replace("{0}", i + "").replace("{1}",
                    change(cellw * i) + ""));
        }
//		sbForWidth.append(HTemplate.replace("{0}", baseW + "").replace("{1}",
//				w + ""));
        sbForWidth.append("</resources>");

        StringBuffer sbForHeight = new StringBuffer();
        sbForHeight.append("<?xml version=\"1.0\" encoding=\"utf-8\"?>\n");
        sbForHeight.append("<resources>");
        float cellh = h * 1.0f / baseH;
        System.out.println("height : " + h + "," + baseH + "," + cellh);
        for (int i = 1; i <= max; i++) {
            sbForHeight.append(WTemplate.replace("{0}", i + "").replace("{1}",
                    change(cellh * i) + ""));
        }
//		sbForHeight.append(WTemplate.replace("{0}", baseH + "").replace("{1}",
//				h + ""));
        sbForHeight.append("</resources>");

        File fileDir = new File(dirStr + File.separator
                + VALUE_TEMPLATE_LAND.replace("{0}", h + "")//
                .replace("{1}", w + ""));
        fileDir.mkdir();

        File layxFile = new File(fileDir.getAbsolutePath(), "lay_x.xml");
        File layyFile = new File(fileDir.getAbsolutePath(), "lay_y.xml");
        try {
            PrintWriter pw = new PrintWriter(new FileOutputStream(layyFile));
            pw.print(sbForWidth.toString());
            pw.close();
            pw = new PrintWriter(new FileOutputStream(layxFile));
            pw.print(sbForHeight.toString());
            pw.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }

    private void generateXmlFile(int w, int h, int max) {

        StringBuffer sbForWidth = new StringBuffer();
        sbForWidth.append("<?xml version=\"1.0\" encoding=\"utf-8\"?>\n");
        sbForWidth.append("<resources>");
        float cellw = w * 1.0f / baseW;

        System.out.println("width : " + w + "," + baseW + "," + cellw);
        for (int i = 1; i <= max; i++) {
            sbForWidth.append(WTemplate.replace("{0}", i + "").replace("{1}",
                    change(cellw * i) + ""));
        }
//		sbForWidth.append(WTemplate.replace("{0}", baseW + "").replace("{1}",
//				w + ""));
        sbForWidth.append("</resources>");

        StringBuffer sbForHeight = new StringBuffer();
        sbForHeight.append("<?xml version=\"1.0\" encoding=\"utf-8\"?>\n");
        sbForHeight.append("<resources>");
        float cellh = h * 1.0f / baseH;
        System.out.println("height : " + h + "," + baseH + "," + cellh);
        for (int i = 1; i <= max; i++) {
            sbForHeight.append(HTemplate.replace("{0}", i + "").replace("{1}",
                    change(cellh * i) + ""));
        }
//		sbForHeight.append(HTemplate.replace("{0}", baseH + "").replace("{1}",
//				h + ""));
        sbForHeight.append("</resources>");

        File fileDir = new File(dirStr + File.separator
                + VALUE_TEMPLATE.replace("{0}", h + "")//
                .replace("{1}", w + ""));
        fileDir.mkdir();

        File layxFile = new File(fileDir.getAbsolutePath(), "lay_x.xml");
        File layyFile = new File(fileDir.getAbsolutePath(), "lay_y.xml");
        try {
            PrintWriter pw = new PrintWriter(new FileOutputStream(layxFile));
            pw.print(sbForWidth.toString());
            pw.close();
            pw = new PrintWriter(new FileOutputStream(layyFile));
            pw.print(sbForHeight.toString());
            pw.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }

    public static float change(float a) {
        int temp = (int) (a * 100);
        return temp / 100f;
    }

    public static float changedp(int num, float scale) {
        return num / scale;
    }

    public static void main(String[] args) {
        int baseW = 1200;
        int baseH = 1920;
        String addition = "";
        try {
            if (args.length >= 3) {
                baseW = Integer.parseInt(args[0]);
                baseH = Integer.parseInt(args[1]);
                addition = args[2];
            } else if (args.length >= 2) {
                baseW = Integer.parseInt(args[0]);
                baseH = Integer.parseInt(args[1]);
            } else if (args.length >= 1) {
                addition = args[0];
            }
        } catch (NumberFormatException e) {

            System.err
                    .println("right input params : java -jar xxx.jar width height w,h_w,h_..._w,h;");
            e.printStackTrace();
            System.exit(-1);
        }

        new GenerateValueFiles(baseW, baseH, addition).generate();
    }

}
```

[今日头条屏幕适配方案终极版，一个极低成本的 Android 屏幕适配方案](https://github.com/JessYanCoding/AndroidAutoSize)

