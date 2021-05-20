### 环境覆盖率-全量代码分析

1.  下载代码到本地，并编译代码。

    ```java
    public void triggerEnvCov(EnvCoverRequest envCoverRequest) {
        //...中间代码省略....
        // 设置状态为待执行
        coverageReport.setRequestStatus(Constants.JobStatus.WAITING.val());
        coverageReportDao.insertCoverageReportById(coverageReport);
        deployInfoDao.insertDeployId(envCoverRequest.getUuid(), envCoverRequest.getAddress(), envCoverRequest.getPort());
        new Thread(() -> {
            // checkout 代码 并且编译 mvn clean compile
            cloneAndCompileCode(coverageReport);
            if (coverageReport.getRequestStatus() != Constants.JobStatus.COMPILE_DONE.val()) {
                log.info("{}计算覆盖率具体步骤...编译失败uuid={}", Thread.currentThread().getName(), coverageReport.getUuid());
                return;
            }
            // 判断是否是增量覆盖率统计
            if (coverageReport.getType() == Constants.ReportType.DIFF.val()) {
                // 计算增量代码中的方法
                calculateDeployDiffMethods(coverageReport);
                if (coverageReport.getRequestStatus() != Constants.JobStatus.DIFF_METHOD_DONE.val()) {
                    log.info("{}计算覆盖率具体步骤...计算增量代码失败，uuid={}", Thread.currentThread().getName(), coverageReport.getUuid());
                    return;
                }
            }
            // 核心方法，统计环境覆盖率，生存报告
            calculateEnvCov(coverageReport);
        }).start();
    ```

    

2.  下载代码到本地codeCloneExecutor.cloneCode(coverageReport);：使用Git进行checkout代码到本地/root/app/super_jacoco/clonecode/**uuid**/nowVersion/路径下。/root/app/super_jacoco/clonecode/**032c1e70-5d4a-4fac-95fa-f245e99a4088**/c906c4ff0c8c3a1085ce7befdc2f34d60b120b5e

3.  编译本地下载的代码：codeCompilerExecutor.compileCode(coverageReport);

    ```shell
    bash -c cd /root/app/super_jacoco/clonecode/032c1e70-5d4a-4fac-95fa-f245e99a4088/c906c4ff0c8c3a1085ce7befdc2f34d60b120b5e&&mvn clean compile >>/root/report/logs/032c1e70-5d4a-4fac-95fa-f245e99a4088.log
    ```

4.  获取到路径下的pom文件，/root/app/super_jacoco/clonecode/032c1e70-5d4a-4fac-95fa-f245e99a4088/c906c4ff0c8c3a1085ce7befdc2f34d60b120b5e/pom.xml

5.  获取pom中有效的modules列表，ArrayList<String> moduleList = MavenModuleUtil.getValidModules(pomPath)。使用正则匹配到所有模块，moduleregex = "<modules>.*?</modules>";

6.  从项目机器上拉取功能测试的执行轨迹.exec文件，计算增量方法覆盖率。calculateEnvCov(coverageReport);

    ```shell
    bash -c cd /root/app/super_jacoco/clonecode/032c1e70-5d4a-4fac-95fa-f245e99a4088/c906c4ff0c8c3a1085ce7befdc2f34d60b120b5e&&java -jar /root/org.jacoco.cli-1.0.2-SNAPSHOT-nodeps.jar dump --address 192.168.173.33 --port 18513 --destfile ./jacoco.exec
    ```

    1.  判断上面的命令是否执行成功，因为java -jar --address的时候可能被测应用连接不上。如果执行失败则结束报告统计。

        ```java
        // Miller: 结束本次统计覆盖率，需要重新启动统计覆盖率
        coverageReport.setErrMsg("获取jacoco.exec 文件失败");
        coverageReport.setRequestStatus(Constants.JobStatus.ENVREPORT_FAIL.val());
        log.error("uuid={}", coverageReport.getUuid(), coverageReport.getErrMsg());
        // Miller:删除下载的代码和生成的jacoco.exec文件
        FileUtil.cleanDir(new File(coverageReport.getNowLocalPath()).getParent());
        ```

    2.  如果dump命令执行成功exitCode == 0, 先删除报告

        ```java
        if (exitCode == 0) {
            CmdExecutor.executeCmd(new String[]{"rm -rf " + REPORT_PATH + coverageReport.getUuid()}, CMD_TIMEOUT);
            String[] moduleList = deployInfoEntity.getChildModules().split(",");
            StringBuilder builder = new StringBuilder("java -jar " + JACOCO_PATH + " report " + deployInfoEntity.getCodePath() + "/jacoco.exec ");
        ```

        

        ```shell
        1. 先删除报告
        bash -c rm -rf /root/report/032c1e70-5d4a-4fac-95fa-f245e99a4088
        2. 生成报告文件
        java -jar /root/org.jacoco.cli-1.0.2-SNAPSHOT-nodeps.jar report /root/app/super_jacoco/clonecode/032c1e70-5d4a-4fac-95fa-f245e99a4088/c906c4ff0c8c3a1085ce7befdc2f34d60b120b5e/jacoco.exec 
        ```

    3.  如果dump命令执行失败exitCode == 1, 则直接结束覆盖率统计，因为可能连接不上服务器了，或者应用重新部署了。如果应用重新部署之后希望重新统计那么就需要新开启一个环境覆盖率统计，如果希望继续统计，则不要调用接口新开启一个环境统计，使用原来的uuid进行获取最新统计结果即可。

        ```java
        if (exitCode == 0){...省略代码}
        else {
            // Miller: 结束本次统计覆盖率，需要重新启动统计覆盖率
            coverageReport.setErrMsg("获取jacoco.exec 文件失败");
            coverageReport.setRequestStatus(Constants.JobStatus.ENVREPORT_FAIL.val());
            log.error("uuid={}", coverageReport.getUuid(), coverageReport.getErrMsg());
            // Miller:删除下载的代码和生成的jacoco.exec文件
            FileUtil.cleanDir(new File(coverageReport.getNowLocalPath()).getParent());
        }
        ```

        

    4.  判断是否多模块，如果多模块需要添加多个参数--sourcefiles和--classfiles 

    5.  判断是否是增量，如果是增量则添加参数-diffFile 方法名称

    6.  最后在命令上追加生存报告的参数,builder.append(" --html ./jacocoreport/ --encoding utf-8 --name " + reportName + ">>" + logFile);

    7.  执行命令生成报告，将每个模块的执行结果都放到当前新建的jacocoreport目录下，这里可以发现因为是多模块所以会有多个参数--sourcefiles和--classfiles ，

        ```shell
        bash -c cd /root/app/super_jacoco/clonecode/032c1e70-5d4a-4fac-95fa-f245e99a4088/c906c4ff0c8c3a1085ce7befdc2f34d60b120b5e
        java -jar /root/org.jacoco.cli-1.0.2-SNAPSHOT-nodeps.jar report /root/app/super_jacoco/clonecode/032c1e70-5d4a-4fac-95fa-f245e99a4088/c906c4ff0c8c3a1085ce7befdc2f34d60b120b5e/jacoco.exec --sourcefiles ./entity/src/main/java/ --classfiles ./entity/target/classes/com/ --sourcefiles ./web/src/main/java/ --classfiles ./web/target/classes/com/ --sourcefiles ./service/src/main/java/ --classfiles ./service/target/classes/com/ --sourcefiles ./mapper/src/main/java/ --classfiles ./mapper/target/classes/com/  --html ./jacocoreport/ --encoding utf-8 --name ManualCoverage>>/root/report/logs/032c1e70-5d4a-4fac-95fa-f245e99a4088.log
        ```

    8.  获取报告路径，/root/app/super_jacoco/clonecode/032c1e70-5d4a-4fac-95fa-f245e99a4088/c906c4ff0c8c3a1085ce7befdc2f34d60b120b5e/jacocoreport/index.html

    9.  如果执行命令java -jar /root/org.jacoco.cli-1.0.2-SNAPSHOT-nodeps.jar report 成功，并且报告已存在，则将报告拷贝目录拷贝/root/report/uuid/目录下

        ```java
        bash -c cp -rf /root/app/super_jacoco/clonecode/032c1e70-5d4a-4fac-95fa-f245e99a4088/c906c4ff0c8c3a1085ce7befdc2f34d60b120b5e/jacocoreport /root/report/032c1e70-5d4a-4fac-95fa-f245e99a4088/
        ```

        -   代码如下covExitCode == 0

            ```java
            if (covExitCode == 0 && reportFile.exists()) {
                try {
                    // 解析并获取覆盖率
                    Document doc = Jsoup.parse(reportFile.getAbsoluteFile(), "UTF-8", "");
                    Elements bars = doc.getElementById("coveragetable").getElementsByTag("tfoot").first().getElementsByClass("bar");
                    Elements lineCtr1 = doc.getElementById("coveragetable").getElementsByTag("tfoot").first().getElementsByClass("ctr1");
                    Elements lineCtr2 = doc.getElementById("coveragetable").getElementsByTag("tfoot").first().getElementsByClass("ctr2");
                    double lineCoverage = 100;
                    double branchCoverage = 100;
                    // 以上这里初始化都换成了1
                    if (doc != null && bars != null) {
                        float lineNumerator = Float.valueOf(lineCtr1.get(1).text().replace(",", ""));
                        float lineDenominator = Float.valueOf(lineCtr2.get(3).text().replace(",", ""));
                        lineCoverage = (lineDenominator - lineNumerator) / lineDenominator * 100;
                        String[] branch = bars.get(1).text().split(" of ");
                        float branchNumerator = Float.valueOf(branch[0].replace(",", ""));
                        float branchDenominator = Float.valueOf(branch[1].replace(",", ""));
                        if (branchDenominator > 0.0) {
                            branchCoverage = (branchDenominator - branchNumerator) / branchDenominator * 100;
                        }
                    }
                    // 复制report报告
                    String[] cppCmd = new String[]{"cp -rf " + reportFile.getParent() + " " + REPORT_PATH + coverageReport.getUuid() + "/"};
                    CmdExecutor.executeCmd(cppCmd, CMD_TIMEOUT);
                    coverageReport.setReportUrl(LocalIpUtils.getTomcatBaseUrl() + coverageReport.getUuid() + "/index.html");
                    coverageReport.setRequestStatus(Constants.JobStatus.SUCCESS.val());
                    coverageReport.setLineCoverage(lineCoverage);
                    coverageReport.setBranchCoverage(branchCoverage);
                    return;
                } catch (Exception e) {
                    coverageReport.setRequestStatus(Constants.JobStatus.ENVREPORT_FAIL.val());
                    coverageReport.setErrMsg("解析jacoco报告失败");
                    log.error("uuid={}", coverageReport.getUuid(), e.getMessage());
                }
            }else{查看下面一个步骤的else分析文档} 
            ```

            

    10.   如果执行java -jar /root/org.jacoco.cli-1.0.2-SNAPSHOT-nodeps.jar report 命令失败 或 报告文件index.html不存在（/jacocoreport/index.html）.将状态改为ENVREPORT_FAIL(212, "统计功能测试增量覆盖率失败"),然后会认为这个是一个父子项目的项目，会分开统计，然后手动合并报告。

          ```shell
          bash -c cd /root/app/super_jacoco/clonecode/8e4b1959-9986-4f2d-a9b2-478f9d1d15a1/c906c4ff0c8c3a1085ce7befdc2f34d60b120b5ejava -jar /root/org.jacoco.cli-1.0.2-SNAPSHOT-nodeps.jar report ./jacoco.exec --sourcefiles ./entity/src/main/java/ --classfiles ./entity/target/classes/com/ --html jacocoreport/entity --encoding utf-8 --name ManualCoverage>>/root/report/logs/8e4b1959-9986-4f2d-a9b2-478f9d1d15a1.log
          ```

          -   第二次执行

              ```shell
              bash -c cd /root/app/super_jacoco/clonecode/8e4b1959-9986-4f2d-a9b2-478f9d1d15a1/c906c4ff0c8c3a1085ce7befdc2f34d60b120b5e&&java -jar /root/org.jacoco.cli-1.0.2-SNAPSHOT-nodeps.jar report ./jacoco.exec --sourcefiles ./mapper/src/main/java/ --classfiles ./mapper/target/classes/com/ --html jacocoreport/mapper --encoding utf-8 --name ManualCoverage>>/root/report/logs/8e4b1959-9986-4f2d-a9b2-478f9d1d15a1.log
              ```

              

          -   区别在于仍为是多项目聚合的项目，每个项目单独生存测试报告，然后在合并在一起。一种情况是一个项目多个moudle那么报告直接聚合在一起参数为"--html ./jacocoreport/"，而现在这种情况是参数为"--html jacocoreport/entity"，单独循环生成报告，然后在手动合并报告。

              ```shell
              java -jar /root/org.jacoco.cli-1.0.2-SNAPSHOT-nodeps.jar report /root/app/super_jacoco/clonecode/032c1e70-5d4a-4fac-95fa-f245e99a4088/c906c4ff0c8c3a1085ce7befdc2f34d60b120b5e/jacoco.exec --sourcefiles ./entity/src/main/java/ --classfiles ./entity/target/classes/com/ --sourcefiles ./web/src/main/java/ --classfiles ./web/target/classes/com/ --sourcefiles ./service/src/main/java/ --classfiles ./service/target/classes/com/ --sourcefiles ./mapper/src/main/java/ --classfiles ./mapper/target/classes/com/  --html ./jacocoreport/ --encoding utf-8 --name ManualCoverage
              ```

              

          -   代码如下covExitCode == -1或报告文件不存在

          ```java
          if (covExitCode == 0 && reportFile.exists()){...}else {
              //Miller: 如果执行java -jar /root/org.jacoco.cli-1.0.2-SNAPSHOT-nodeps.jar report 命令失败 或 报告不存在
              coverageReport.setRequestStatus(Constants.JobStatus.ENVREPORT_FAIL.val());
              // 可能不同子项目存在同一类名
              int littleExitCode = 0;
              ArrayList<String> childReportList = new ArrayList<>();
              for (String module : moduleList) {
                  StringBuilder buildertmp = new StringBuilder("java -jar " + JACOCO_PATH + " report ./jacoco.exec");
                  buildertmp.append(" --sourcefiles ./" + module + "/src/main/java/");
                  buildertmp.append(" --classfiles ./" + module + "/target/classes/com/");
                  if (!StringUtils.isEmpty(coverageReport.getDiffMethod())) {
                      builder.append("--diffFile " + coverageReport.getDiffMethod());
                  }
                  buildertmp.append(" --html jacocoreport/" + module + " --encoding utf-8 --name " + reportName + ">>" + logFile);
                  littleExitCode += CmdExecutor.executeCmd(new String[]{"cd " + deployInfoEntity.getCodePath() + "&&" + buildertmp.toString()}, CMD_TIMEOUT);
                  if (littleExitCode == 0) {
                      childReportList.add(deployInfoEntity.getCodePath() + "/jacocoreport/" + module + "/index.html");
                  }
              }
              if (littleExitCode == 0) {
                  // 合并多份报告
                  CmdExecutor.executeCmd(new String[]{"cd " + deployInfoEntity.getCodePath() + "&&cp -rf jacocoreport " + REPORT_PATH + coverageReport.getUuid() + "/"}, CMD_TIMEOUT);
                  // 合并html文件
                  Integer[] result = MergeReportHtml.mergeHtml(childReportList, REPORT_PATH + coverageReport.getUuid() + "/index.html");
          
                  if (result[0] > 0) {
                      coverageReport.setReportUrl(LocalIpUtils.getTomcatBaseUrl() + coverageReport.getUuid() + "/index.html");
                      coverageReport.setRequestStatus(Constants.JobStatus.SUCCESS.val());
                      FileUtil.cleanDir(new File(coverageReport.getNowLocalPath()).getParent());
                      //cp -r /app/diff_code_coverage/resource/jacoco-resources /root/report/8e4b1959-9986-4f2d-a9b2-478f9d1d15a1
                      CmdExecutor.executeCmd(new String[]{"cp -r " + JACOCO_RESOURE_PATH + " " + REPORT_PATH + coverageReport.getUuid()}, CMD_TIMEOUT);
                      coverageReport.setLineCoverage(Double.valueOf(result[2]));
                      coverageReport.setBranchCoverage(Double.valueOf(result[1]));
                      return;
                  } else {
                      coverageReport.setRequestStatus(Constants.JobStatus.ENVREPORT_FAIL.val());
                      coverageReport.setErrMsg("生成jacoco报告失败 ");
                  }
              } else {
                  // 生成报告错误
                  coverageReport.setRequestStatus(Constants.JobStatus.ENVREPORT_FAIL.val());
                  coverageReport.setErrMsg("生成jacoco报告失败");
              }
          }
          ```

          -   第21行代码获取到的所有子项目列表路径

              ```java
               size = 4
                   0 = "/root/app/super_jacoco/clonecode/8e4b1959-9986-4f2d-a9b2-478f9d1d15a1/c906c4ff0c8c3a1085ce7befdc2f34d60b120b5e/jacocoreport/entity/index.html"
                   1 = "/root/app/super_jacoco/clonecode/8e4b1959-9986-4f2d-a9b2-478f9d1d15a1/c906c4ff0c8c3a1085ce7befdc2f34d60b120b5e/jacocoreport/web/index.html"
                   2 = "/root/app/super_jacoco/clonecode/8e4b1959-9986-4f2d-a9b2-478f9d1d15a1/c906c4ff0c8c3a1085ce7befdc2f34d60b120b5e/jacocoreport/service/index.html"
                   3 = "/root/app/super_jacoco/clonecode/8e4b1959-9986-4f2d-a9b2-478f9d1d15a1/c906c4ff0c8c3a1085ce7befdc2f34d60b120b5e/jacocoreport/mapper/index.html"
              ```

    ### 环境覆盖率，计算增量代码中的方法列表

    1.  执行时判断是否需要计算增量覆盖率。

        ```java
        public void triggerEnvCov(EnvCoverRequest envCoverRequest) {
            coverageReport.setRequestStatus(Constants.JobStatus.WAITING.val());
            coverageReportDao.insertCoverageReportById(coverageReport);
            deployInfoDao.insertDeployId(envCoverRequest.getUuid(), envCoverRequest.getAddress(), envCoverRequest.getPort());
            new Thread(() -> {
                cloneAndCompileCode(coverageReport);
                if (coverageReport.getRequestStatus() != Constants.JobStatus.COMPILE_DONE.val()) {
                    log.info("{}计算覆盖率具体步骤...编译失败uuid={}", Thread.currentThread().getName(), coverageReport.getUuid());
                    return;
                }
                // 判断是否是增量覆盖率统计
                if (coverageReport.getType() == Constants.ReportType.DIFF.val()) {
                    // 计算增量代码中的方法
                    calculateDeployDiffMethods(coverageReport);
                    if (coverageReport.getRequestStatus() != Constants.JobStatus.DIFF_METHOD_DONE.val()) {
                        log.info("{}计算覆盖率具体步骤...计算增量代码失败，uuid={}", Thread.currentThread().getName(), coverageReport.getUuid());
                        return;
                    }
                }
                calculateEnvCov(coverageReport);
            }).start();
        ```

        

    2.  获取增量覆盖率

    ```java
    // 下载代码并计算diff方法
    public void executeDiffMethods(CoverageReportEntity coverageReport) {
        StringBuffer diffFile = new StringBuffer();
        long s = System.currentTimeMillis();
        // 获取增量代码方法
        HashMap map = JDiffFiles.diffMethodsListNew(coverageReport);
        if (!CollectionUtils.isEmpty(map)) {
            Iterator<Map.Entry<String, String>> iterator = map.entrySet().iterator();
            while (iterator.hasNext()) {
                Map.Entry<String, String> entry = iterator.next();
                diffFile.append(entry.getKey() + ":" + entry.getValue() + "%");
            }
            coverageReport.setDiffMethod(diffFile.toString());
        }
    
        log.info("uuid={} 增量计算耗时：{}", coverageReport.getUuid(), (System.currentTimeMillis() - s));
    }
    ```

    3.  统计两个版本之间的增量覆盖率方法列表

    ```java
    public static HashMap<String, String> diffMethodsListNew(CoverageReportEntity coverageReport) {
    
        List<DiffEntry> diff = newGit.diff().setOldTree(oldTree).setNewTree(newTree).setShowNameAndStatusOnly(true).call();
        for (DiffEntry diffEntry : diff) {
            // 判断变更是否属于删除
            if (diffEntry.getChangeType() == DiffEntry.ChangeType.DELETE) {
                continue;
            }
            if (diffEntry.getNewPath().indexOf(coverageReport.getSubModule()) < 0) {
                continue;
            }
            if(diffEntry.getNewPath().indexOf("src/test/java")!=-1){
                continue;
            }
            if (diffEntry.getNewPath().endsWith(".java")) {
                String nowclassFile = diffEntry.getNewPath();
                // 判断变更是否属于新增
                if (diffEntry.getChangeType() == DiffEntry.ChangeType.ADD) {
                    map.put(nowclassFile.replace(".java", ""), "true");
                } else if (diffEntry.getChangeType() == DiffEntry.ChangeType.MODIFY) {
                    // 判断变更是否属于修改
                    MethodParser methodParser = new MethodParser();
                    // 获取 baseVersion 中的方法列表
                    HashMap<String, String> baseMMap = methodParser.parseMethodsMd5(oldGit.getRepository().getDirectory().getParent() + "/" + nowclassFile);
                    // 获取 nowVersion 中的方法列表
                    HashMap<String, String> nowMMap = methodParser.parseMethodsMd5(newGit.getRepository().getDirectory().getParent() + "/" + nowclassFile);
                    // 比较两个方法列表的区别，找出修改的方法
                    HashMap<String, String> resMap = diffMethods(baseMMap, nowMMap);
                    if (resMap.isEmpty()) {
                        continue;
                    } else {
                        StringBuilder builder = new StringBuilder("");
                        for (String v : resMap.values()) {
                            builder.append(v + "#");
                        }
                        map.put(nowclassFile.replace(".java", ""), builder.toString());
                    }
                }
                //其他变更类型就不处理了。比如 RENAME、COPY
            }
        }
        if (map.isEmpty()) {
            coverageReport.setLineCoverage((double) 100);
            coverageReport.setBranchCoverage((double) 100);
            coverageReport.setRequestStatus(Constants.JobStatus.SUCCESS.val());
            coverageReport.setReportUrl(Constants.NO_DIFFCODE_REPORT);
            // 删除下载的代码
            FileUtil.cleanDir(new File(coverageReport.getNowLocalPath()).getParent());
            coverageReport.setErrMsg("没有增量代码");
        } else {
            coverageReport.setRequestStatus(Constants.JobStatus.DIFF_METHOD_DONE.val());
        }
        return map;
    }
    ```

    

