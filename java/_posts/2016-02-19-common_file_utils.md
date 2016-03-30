---
layout: post
title: "开发中文件处理工具"
modified: 2016-1-23 18:10:00
excerpt: "开发中一些对文件处理的小工具（重命名、正则匹配），Java代码实现......"
tags: [Java]
comments: true
---
前言：在开发中经常要处理一些文件，比如美工给切图是依照iOS的规范切的，切图的文件名都叫（XXX@2x.png）或（XXX@3x.png）,然而身为Android开发的程序猿并不需要@2x和@3x这样的东西，怎么办？如果按F2一个个重名命，做一些简单重复的工作也太不符合我等程序猿的逼格了吧。这个时候就会想着写一段小代码来自动来重命名这些文件，这样岂不是快哉。  
好了，下面贴出一些我平时处理这种事时的代码，以后如果有更新也会贴上来。

**1，重命名包含@2x和@3x这样的图片**

	/**
	 * 移除文件名中的TAG，并移动文件到根目录下的TAG文件夹下
	 * TAG表示要替换的文件关键字，重命名时会去掉TAG
	 * 
	 * 解决问题：设计师给出的切图都是针对iOS，文件名都叫（XXX@2x.png）或（XXX@3x.png），Android使用时需要移除@2x或@3x这样的TAG，并把他们移动到不同的文件夹下
	 * @author iOnesmile
	 *
	 */
	public class AllotFileTest {
		private static final String BASE_PATH = "C:\\Users\\Administrator\\Desktop\\icon";
		private static final String TAG = "@2x";
	
		public static void main(String[] args) {
			allotAndRename(BASE_PATH, TAG);
		}
	
		public static void allotAndRename(String basePath, String point) {
			File baseFile = new File(basePath);
			File targetFile = new File(basePath + "\\" + point);
			if (!targetFile.exists()) {
				targetFile.mkdirs();
			}
			File[] sourceFiles = baseFile.listFiles();
			for (File item : sourceFiles) {
				if (item.isFile()) {
					String childName = item.getName();
					if (childName.contains(point)) {
						System.out.println("  --- " + childName);
						childName = childName.replace(point, "");
						// 重命名文件，在不同文件夹下会移动文件
						item.renameTo(new File(targetFile, childName));
					}
				}
			}
		}
	}



**2，快速生成Drawable/Selector文件**

	/**
	 * 根据切图名，生成一个切图的Selector文件
	 * 如：有两张切图，ic_lamp_nor.png和ic_lamp_down.png，执行以下代码后就会生成一个selector_ic_lamp.xml的文件
	 * 
	 * @author iOnesmile
	 *
	 */
	public class SeletorTest {
	
		private static final String IMG_SUFFIX = ".png";
		private static final String ACTIVE_TAG = "_down";
		private static final String NORMAL_TAG = "_nor";
		private static final String MODEL_DOC = "Model_Press.txt";	// Model文件名称，替换该Model的关键内容
		
		
		// AutoGene\Model_Check.txt
		/*
		 
		<?xml version="1.0" encoding="utf-8"?>
		<selector xmlns:android="http://schemas.android.com/apk/res/android">
		    <item android:drawable="@drawable/[CheckedIMG]" android:state_checked="true"></item>
		    <item android:drawable="@drawable/[NormalIMG]"></item>
		</selector>
		 
		 */
		
		// AutoGene\Model_Press.txt
		/*
		 
		<?xml version="1.0" encoding="utf-8"?>
		<selector xmlns:android="http://schemas.android.com/apk/res/android">
		    <item android:drawable="@drawable/[CheckedIMG]" android:state_pressed="true"></item>
		    <item android:drawable="@drawable/[NormalIMG]"></item>
		</selector>
		 
		 */
		
		public static final String MODEL_FILE_PAT = "C:\\Users\\Administrator\\Desktop\\AutoGene\\" + MODEL_DOC;
		private static final String IMAGE_SRC_PATH = "C:\\Users\\Administrator\\Desktop\\AutoGene\\ImageSrc";
		private static final String IMAGE_TARGET_PATH = "C:\\Users\\Administrator\\Desktop\\AutoGene\\TargetImage";
	
		public static void main(String[] args) throws IOException {
			// 获取Model文件文本
			String modelText = getModeText(MODEL_FILE_PAT);
			System.out.println(modelText);
			
			// 获取图片目录下所有文件名
			String[] fileNames = new File(IMAGE_SRC_PATH).list();
			
			// 截取文件名的关键Key
			String[] baseFileName = getBaseFileName(fileNames); 
			System.out.println(Arrays.toString(baseFileName));
			
			// 将Model内的内容替换关键Key，然后生成新的文件
			for (String fileName : baseFileName) {
				String targetContent = modelText;
				String checkedText = fileName + ACTIVE_TAG;
				String normalText = fileName + NORMAL_TAG;
				targetContent = targetContent.replace("[CheckedIMG]", checkedText);
				targetContent = targetContent.replace("[NormalIMG]", normalText);
				writeFile(fileName, targetContent);
			}
		}
	
		// 写文件，将String字符串生成一个文件
		private static void writeFile(String fileName, String targetContent) throws IOException {
			File writeFile = new File(IMAGE_TARGET_PATH, "selector_" + fileName + ".xml");
			BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(new FileOutputStream(writeFile)));
			bw.write(targetContent);
			bw.close();
		}
	
		// 将文件名字符串数组如XXX_nor.png转换成XXX的字符串数组
		private static String[] getBaseFileName(String[] fileNames) {
			ArrayList<String> list = new ArrayList<String>();
			String suffix = ACTIVE_TAG + IMG_SUFFIX;
			for (String item : fileNames) {
				if (item.endsWith(suffix)) {
					String name = item.replace(suffix, "");
					if (!name.equals(item)) {
						list.add(name);
					}
				}
			}
			return list.toArray(new String[list.size()]);
		}
	
		// 读文件，将文本文件转换成String字符串
		private static String getModeText(String modelFilePat) throws IOException {
			BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream(new File(modelFilePat))));
			StringBuilder sb = new StringBuilder();
			String line = "";
			while ((line = br.readLine()) != null) {
				sb.append(line).append("\r\n");
			}
			br.close();
			return sb.toString();
		}
	}

