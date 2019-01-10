---
layout: post
title:  "新个税计算"
date:   2019-01-10 22:45:52
categories: 生活 工具
published: true
comments: true
thread: 20181213224555555
---
新个税计算-Java实现
---
> [参考](https://gerensuodeshui.cn)

```java
public class TaxCal {

    /**
     * 计算 month 个月每个月实际收入
     *
     * @param context
     * @param month
     */
    public static void cal(TaxContext context, int month) {

        if (month > 12 || month < 1) {
            throw new IllegalArgumentException("");
        }

        for (int i = 1; i <= month; i++) {
            // 累计
            context.plusOneMonthRegular();
            // 应税额度
            double needTaxAmount = context.getAccumulativeIncome() - context.getAccumulativeDeduct();

            // 计算应税率和累计扣除数
            double rate = context.getTaxRate(needTaxAmount);
            double deduct = context.getTaxDeduct(needTaxAmount);

            // 计算出应缴税
            double monthTaxLogical = needTaxAmount * rate - deduct;
            // 计算出实缴税
            double monthTaxPhysical = monthTaxLogical - context.getLastMonthTax();
            // 记录本月纳税额度
            context.logLastMonthTax(monthTaxLogical);

            context.getMap().put(i, context.getConfig().getIncome() - monthTaxPhysical - context.getConfig().getSocialAssure());
            System.out.println(i + "月份：缴税扣除" + monthTaxPhysical + "元， 社保扣除"
                    + context.getConfig().getSocialAssure() + ", 到手" + context.getMap().get(i));
        }
    }

    public static void main(String[] args) {
        TaxConfig config = new TaxConfig(20000, 5000, 3744);
        TaxContext context = new TaxContext(config);
        cal(context, 12);

        double accumulativeAmount = 0;
        for (Map.Entry<Integer, Double> entry : context.getMap().entrySet()) {
            accumulativeAmount += entry.getValue();
        }

        System.out.println(accumulativeAmount);

    }

}



public class TaxConfig {
    /** 每月收入 */
    private double income;
    /** 免税基数 */
    private double basic;
    /** 社保*/
    private double socialAssure;

    public TaxConfig(double income, double basic, double socialAssure) {
        this.income = income;
        this.basic = basic;
        this.socialAssure = socialAssure;
    }


    public double getIncome() {
        return income;
    }

    public double getBasic() {
        return basic;
    }

    public void setIncome(double income) {
        this.income = income;
    }

    public double getSocialAssure() {
        return socialAssure;
    }

    public void setSocialAssure(double socialAssure) {
        this.socialAssure = socialAssure;
    }
}


public class TaxContext {
    /** 累计收入 */
    private double accumulativeIncome;
    /** 累计扣除 */
    private double accumulativeDeduct;
    /** 上个月应交税 */
    private double lastMonthTax;
    private TaxConfig config;
    private HashMap<Integer, Double> map = new HashMap<>();

    public TaxContext(TaxConfig config) {
        this.config = config;
    }

    public HashMap<Integer, Double> getMap() {
        return map;
    }

    /**
     * 累计
     */
    public void plusOneMonthRegular() {
        accumulativeIncome+=config.getIncome();
        accumulativeDeduct+=config.getSocialAssure();
        accumulativeDeduct+=config.getBasic();
    }

    public void logLastMonthTax(double amount) {
        lastMonthTax= amount;
    }

    public double getLastMonthTax() {
        return lastMonthTax;
    }

    public double getAccumulativeIncome() {
        return accumulativeIncome;
    }

    public double getAccumulativeDeduct() {
        return accumulativeDeduct;
    }

    public TaxConfig getConfig() {
        return config;
    }

    public double getTaxRate(double taxAmount) {
        if (taxAmount <= 36000) {
            return 0.03;
        } else if ( taxAmount > 36000 && taxAmount <= 144000 ) {
            return 0.1;
        }else if ( taxAmount > 144000 && taxAmount <= 300000 ) {
            return 0.2;
        }else if ( taxAmount > 300000 && taxAmount <= 420000 ) {
            return 0.25;
        }else if ( taxAmount > 420000 && taxAmount <= 660000 ) {
            return 0.3;
        } else if (taxAmount > 660000 && taxAmount <= 960000) {
            return 0.35;
        } else {
            return 0.45;
        }
    }


    public double getTaxDeduct(double taxAmount) {
        if (taxAmount <= 36000) {
            return 0;
        } else if ( taxAmount > 36000 && taxAmount <= 144000 ) {
            return 2520;
        }else if ( taxAmount > 144000 && taxAmount <= 300000 ) {
            return 16920;
        }else if ( taxAmount > 300000 && taxAmount <= 420000 ) {
            return 31920;
        }else if ( taxAmount > 420000 && taxAmount <= 660000 ) {
            return 52920;
        } else if (taxAmount > 660000 && taxAmount <= 960000) {
            return 85920;
        } else {
            return 181920;
        }
    }
}
```
