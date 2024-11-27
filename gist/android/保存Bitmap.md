### 保存Bitmap到文件

```java
import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.Bitmap.CompressFormat;
import android.os.Environment;
import com.qq.e.comm.util.GDTLogger;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;

public class BitmapSaveUtil {

    private static final String TAG = "BitmapSaveUtil";

    public static void saveBitmapToFile(Context context, Bitmap bitmap, String dirPath, String fileName) {
        // 获取外部存储目录
        File dir = new File(
                context.getFilesDir().getAbsolutePath() + File.separator + "sample" + File.separator + dirPath);
        if (!dir.exists()) {
            dir.mkdirs();
        }
        // 创建文件对象
        File file = new File(dir, fileName);
        FileOutputStream out = null;
        try {
            out = new FileOutputStream(file);
            // 压缩Bitmap并写入文件
            boolean success = bitmap.compress(CompressFormat.PNG, 100, out); // 100表示最高质量
            if (success) {

            } else {
                GDTLogger.e(TAG + "Failed to save the bitmap.");
            }
        } catch (IOException e) {
            e.printStackTrace();
            GDTLogger.e(TAG + "Error while saving the bitmap.", e);
        } finally {
            if (out != null) {
                try {
                    out.close();
                } catch (IOException e) {
                    e.printStackTrace();
                    GDTLogger.e(TAG + "Error while saving the bitmap.", e);
                }
            }
        }
    }
}
```