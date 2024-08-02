---
title: ES 导入 导出
date: 2017-04-25 10:25
tags: 
  - Elasticsearch
  - Java
categories:
  - [Elasticsearch]
---

# ES 导入 导出

java代码

## 导入

```

public class Input {

    /**
     * 文件保存路径
     */
    private static final String filePath = "F:\\数据文件\\p1\\p1_10.json";

    /**
     * 索引名称
     */
    private static final String indexName = "dwd-p1";

    /**
     * 类型名称
     */
    private static final String typeName = "dwdata";

    public static void main(String[] args) throws UnknownHostException {

        Settings settings = Settings.settingsBuilder().put("cluster.name", "hinge-es").build();
        TransportClient client = TransportClient.builder().settings(settings).build()
                .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("192.168.1.1"), 9300))
                .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("192.168.1.2"), 9300));

        BufferedReader br = null;
        try {
            br = new BufferedReader(new FileReader(new File(filePath)));

            String json;

            int count = 0;
            long total = 0;

            BulkRequestBuilder bulkRequest = client.prepareBulk();

            while ((json = br.readLine()) != null) {
                count++;
                total++;

                bulkRequest.add(client.prepareIndex(indexName, typeName).setSource(json));

                if (count == 500) {
                    BulkResponse bulkResponse = bulkRequest.get();
                    if (bulkResponse.hasFailures()) {
                        System.err.println("############ 出错了！！！！！");
                    }

                    bulkRequest = client.prepareBulk();
                    count = 0;
                    System.out.println("已经导入：" + total);
                }

            }

            if (count != 0) {
                BulkResponse bulkResponse = bulkRequest.get();
                if (bulkResponse.hasFailures()) {
                    System.err.println("############ 出错了！！！！！");
                }

                System.out.println("已经导入：" + total);

            }

            System.out.println("导入结束");

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (br != null) {
                try {
                    br.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }

}
```

## 导出  

```
public class Output {

    /**
     * 一批获取数据
     */
    private static final int BATCH_SIZE = 10000;

    /**
     * 文件记录数
     */
    private static final int FILE_RECORD = 300000;

    /**
     * 文件保存路径
     */
    private static final String filePath = "F:\\数据文件\\dwd-p1\\";

    /**
     * 索引名称
     */
    private static final String indexName = "dwd-p1";

    /**
     * 类型名称
     */
    private static final String typeName = "dwdata";

    public static void main(String[] args) throws UnknownHostException {

        Settings settings = Settings.settingsBuilder().put("cluster.name", "hinge-es").build();
        TransportClient client = TransportClient.builder().settings(settings).build()
                .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("192.168.1.1"), 9300))
                .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("192.168.1.2"), 9300));

        SearchResponse scrollResp = client.prepareSearch(indexName).setTypes(typeName).setQuery(QueryBuilders.matchAllQuery()).setSize(BATCH_SIZE).setScroll(new TimeValue(600000))
                .execute().actionGet();

        int fileIndex = 0;

        String outputFile = getFileName(fileIndex);

        BufferedWriter out = null;

        long totalSize = 0;

        try {

            out = new BufferedWriter(new FileWriter(outputFile));

            while (true) {
                for (SearchHit hit : scrollResp.getHits().getHits()) {

                    totalSize++;

                    out.write(hit.getId());// ID
                    out.write("\r\n");

                    out.write(hit.getSourceAsString());// 数据
                    out.write("\r\n");
                    out.flush();
                }

                System.out.println("已经导出：" + totalSize);

                scrollResp = client.prepareSearchScroll(scrollResp.getScrollId()).setScroll(new TimeValue(60000)).execute().actionGet();

                if (scrollResp.getHits().getHits().length == 0) {
                    break;
                }

                if (totalSize > FILE_RECORD) {
                    out.flush();
                    out.close();
                    out = null;

                    System.out.println(outputFile);
                    System.out.println("导出结束，总共导出数据：" + totalSize);
                    totalSize = 0;
                    fileIndex++;
                    outputFile = getFileName(fileIndex);
                    out = new BufferedWriter(new FileWriter(outputFile));
                }
            }

            System.out.println(outputFile);
            System.out.println("导出结束，总共导出数据：" + totalSize);

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (out != null) {
                try {
                    out.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        System.out.println("查询结束");
    }

    private static String getFileName(int fileIndex) {
        String outputFile = filePath + indexName + "#" + typeName + "_" + fileIndex + ".json";
        return outputFile;
    }

}
```
