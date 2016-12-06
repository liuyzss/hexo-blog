---
title: 等额本息、等额本金利息计算工具
date: 2016-12-06 17:13:38
updated: 2016-12-06 17:13:38
tags:
- java 
- finance
- util
categories:
- java 
- util
---

# 利息计算公式

## 等额本息法： 
每期应还款额＝【借款本金×月利率×（1＋月利率）^还款期数】/【（1＋月利率）^还款期数－1】 
## 等额本金法：
每期应还本金=借款本金/还款期数
每期利息=贷款余额 x 月利率
贷款余额=上期余额 - 每期所还本金

# 等额本息计算工具

``` java
/**
 * Description:等额本息工具类
 * Copyright: Copyright (corporation)2015
 * Company: Corporation
 * @author: liuyang
 * @version: 1.0
 * Created at: 2015年11月30日 下午3:45:46
 * Modification History:
 * Modified by : 
 */

package com.kw.finance;

import java.math.BigDecimal;
import java.util.HashMap;
import java.util.Map;

/**
 * 等额本息还款，也称定期付息，即借款人每月按相等的金额偿还贷款本息，其中每月贷款利息按月初剩余贷款本金计算并逐月结清。把按揭贷款的本金总额与利息总额相加，
 * 然后平均分摊到还款期限的每个月中。作为还款人，每个月还给银行固定金额，但每月还款额中的本金比重逐月递增、利息比重逐月递减。
 */

public class AverageCapitalPlusInterestUtils {

	/**
	 * 等额本息计算获取还款方式为等额本息的每月偿还本金和利息
	 * 
	 * 公式：每月偿还本息=〔贷款本金×月利率×(1＋月利率)＾还款月数〕÷〔(1＋月利率)＾还款月数-1〕
	 * 
	 * @param invest
	 *            总借款额（贷款本金）
	 * @param yearRate
	 *            年利率
	 * @param month
	 *            还款总月数
	 * @return 每月偿还本金和利息,不四舍五入，直接截取小数点最后两位
	 */
	public static double getPerMonthPrincipalInterest(double invest, double yearRate, int totalmonth) {
		double monthRate = yearRate / 12;
		BigDecimal monthIncome = new BigDecimal(invest).multiply(new BigDecimal(monthRate * Math.pow(1 + monthRate, totalmonth))).divide(
				new BigDecimal(Math.pow(1 + monthRate, totalmonth) - 1), 2, BigDecimal.ROUND_DOWN);
		return monthIncome.doubleValue();
	}

	/**
	 * 等额本息计算获取还款方式为等额本息的每月偿还利息
	 * 
	 * 公式：每月偿还利息=贷款本金×月利率×〔(1+月利率)^还款月数-(1+月利率)^(还款月序号-1)〕÷〔(1+月利率)^还款月数-1〕
	 * 
	 * @param invest
	 *            总借款额（贷款本金）
	 * @param yearRate
	 *            年利率
	 * @param month
	 *            还款总月数
	 * @return 每月偿还利息
	 */
	public static Map<Integer, BigDecimal> getPerMonthInterest(double invest, double yearRate, int totalmonth) {
		Map<Integer, BigDecimal> map = new HashMap<Integer, BigDecimal>();
		double monthRate = yearRate / 12;
		BigDecimal monthInterest;
		for (int i = 1; i < totalmonth + 1; i++) {
			BigDecimal multiply = new BigDecimal(invest).multiply(new BigDecimal(monthRate));
			BigDecimal sub = new BigDecimal(Math.pow(1 + monthRate, totalmonth)).subtract(new BigDecimal(Math.pow(1 + monthRate, i - 1)));
			monthInterest = multiply.multiply(sub).divide(new BigDecimal(Math.pow(1 + monthRate, totalmonth) - 1), 6, BigDecimal.ROUND_DOWN);
			monthInterest = monthInterest.setScale(2, BigDecimal.ROUND_DOWN);
			map.put(i, monthInterest);
		}
		return map;
	}

	/**
	 * 等额本息计算获取还款方式为等额本息的每月偿还本金
	 * 
	 * @param invest
	 *            总借款额（贷款本金）
	 * @param yearRate
	 *            年利率
	 * @param month
	 *            还款总月数
	 * @return 每月偿还本金
	 */
	public static Map<Integer, BigDecimal> getPerMonthPrincipal(double invest, double yearRate, int totalmonth) {
		double monthRate = yearRate / 12;
		BigDecimal monthIncome = new BigDecimal(invest).multiply(new BigDecimal(monthRate * Math.pow(1 + monthRate, totalmonth))).divide(
				new BigDecimal(Math.pow(1 + monthRate, totalmonth) - 1), 2, BigDecimal.ROUND_DOWN);
		Map<Integer, BigDecimal> mapInterest = getPerMonthInterest(invest, yearRate, totalmonth);
		Map<Integer, BigDecimal> mapPrincipal = new HashMap<Integer, BigDecimal>();

		for (Map.Entry<Integer, BigDecimal> entry : mapInterest.entrySet()) {
			mapPrincipal.put(entry.getKey(), monthIncome.subtract(entry.getValue()));
		}
		return mapPrincipal;
	}

	/**
	 * 等额本息计算获取还款方式为等额本息的总利息
	 * 
	 * @param invest
	 *            总借款额（贷款本金）
	 * @param yearRate
	 *            年利率
	 * @param month
	 *            还款总月数
	 * @return 总利息
	 */
	public static double getInterestCount(double invest, double yearRate, int totalmonth) {
		BigDecimal count = new BigDecimal(0);
		Map<Integer, BigDecimal> mapInterest = getPerMonthInterest(invest, yearRate, totalmonth);

		for (Map.Entry<Integer, BigDecimal> entry : mapInterest.entrySet()) {
			count = count.add(entry.getValue());
		}
		return count.doubleValue();
	}

	/**
	 * 应还本金总和
	 * 
	 * @param invest
	 *            总借款额（贷款本金）
	 * @param yearRate
	 *            年利率
	 * @param month
	 *            还款总月数
	 * @return 应还本金总和
	 */
	public static double getPrincipalInterestCount(double invest, double yearRate, int totalmonth) {
		double monthRate = yearRate / 12;
		BigDecimal perMonthInterest = new BigDecimal(invest).multiply(new BigDecimal(monthRate * Math.pow(1 + monthRate, totalmonth))).divide(
				new BigDecimal(Math.pow(1 + monthRate, totalmonth) - 1), 2, BigDecimal.ROUND_DOWN);
		BigDecimal count = perMonthInterest.multiply(new BigDecimal(totalmonth));
		count = count.setScale(2, BigDecimal.ROUND_DOWN);
		return count.doubleValue();
	}

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		double invest = 20000; // 本金
		int month = 12;
		double yearRate = 0.15; // 年利率
		double perMonthPrincipalInterest = getPerMonthPrincipalInterest(invest, yearRate, month);
		System.out.println("等额本息---每月还款本息：" + perMonthPrincipalInterest);
		Map<Integer, BigDecimal> mapInterest = getPerMonthInterest(invest, yearRate, month);
		System.out.println("等额本息---每月还款利息：" + mapInterest);
		Map<Integer, BigDecimal> mapPrincipal = getPerMonthPrincipal(invest, yearRate, month);
		System.out.println("等额本息---每月还款本金：" + mapPrincipal);
		double count = getInterestCount(invest, yearRate, month);
		System.out.println("等额本息---总利息：" + count);
		double principalInterestCount = getPrincipalInterestCount(invest, yearRate, month);
		System.out.println("等额本息---应还本息总和：" + principalInterestCount);
	}
}
```

# 等额本金计算工具

``` java
/**
 * Description:等额本金工具类
 * Copyright: Copyright (Corporation)2015
 * Company: Corporation
 * @author: liuyang
 * @version: 1.0
 * Created at: 2015年12月1日 上午8:38:23
 * Modification History:
 * Modified by : 
 */

package com.kw.finance;

import java.math.BigDecimal;
import java.util.HashMap;
import java.util.Map;

/**
 * 等额本金是指一种贷款的还款方式，是在还款期内把贷款数总额等分，每月偿还同等数额的本金和剩余贷款在该月所产生的利息，这样由于每月的还款本金额固定，
 * 而利息越来越少，借款人起初还款压力较大，但是随时间的推移每月还款数也越来越少。
 */
public class AverageCapitalUtils {

	/**
	 * 等额本金计算获取还款方式为等额本金的每月偿还本金和利息
	 * 
	 * 公式：每月偿还本金=(贷款本金÷还款月数)+(贷款本金-已归还本金累计额)×月利率
	 * 
	 * @param invest
	 *            总借款额（贷款本金）
	 * @param yearRate
	 *            年利率
	 * @param month
	 *            还款总月数
	 * @return 每月偿还本金和利息,不四舍五入，直接截取小数点最后两位
	 */
	public static Map<Integer, Double> getPerMonthPrincipalInterest(double invest, double yearRate, int totalMonth) {
		Map<Integer, Double> map = new HashMap<Integer, Double>();
		// 每月本金
		double monthPri = getPerMonthPrincipal(invest, totalMonth);
		// 获取月利率
		double monthRate = yearRate / 12;
		monthRate = new BigDecimal(monthRate).setScale(6, BigDecimal.ROUND_DOWN).doubleValue();
		for (int i = 1; i <= totalMonth; i++) {
			double monthRes = monthPri + (invest - monthPri * (i - 1)) * monthRate;
			monthRes = new BigDecimal(monthRes).setScale(2, BigDecimal.ROUND_DOWN).doubleValue();
			map.put(i, monthRes);
		}
		return map;
	}

	/**
	 * 等额本金计算获取还款方式为等额本金的每月偿还利息
	 * 
	 * 公式：每月应还利息=剩余本金×月利率=(贷款本金-已归还本金累计额)×月利率
	 * 
	 * @param invest
	 *            总借款额（贷款本金）
	 * @param yearRate
	 *            年利率
	 * @param month
	 *            还款总月数
	 * @return 每月偿还利息
	 */
	public static Map<Integer, Double> getPerMonthInterest(double invest, double yearRate, int totalMonth) {
		Map<Integer, Double> inMap = new HashMap<Integer, Double>();
		double principal = getPerMonthPrincipal(invest, totalMonth);
		Map<Integer, Double> map = getPerMonthPrincipalInterest(invest, yearRate, totalMonth);
		for (Map.Entry<Integer, Double> entry : map.entrySet()) {
			BigDecimal principalBigDecimal = new BigDecimal(principal);
			BigDecimal principalInterestBigDecimal = new BigDecimal(entry.getValue());
			BigDecimal interestBigDecimal = principalInterestBigDecimal.subtract(principalBigDecimal);
			interestBigDecimal = interestBigDecimal.setScale(2, BigDecimal.ROUND_DOWN);
			inMap.put(entry.getKey(), interestBigDecimal.doubleValue());
		}
		return inMap;
	}

	/**
	 * 等额本金计算获取还款方式为等额本金的每月偿还本金
	 * 
	 * 公式：每月应还本金=贷款本金÷还款月数
	 * 
	 * @param invest
	 *            总借款额（贷款本金）
	 * @param yearRate
	 *            年利率
	 * @param month
	 *            还款总月数
	 * @return 每月偿还本金
	 */
	public static double getPerMonthPrincipal(double invest, int totalMonth) {
		BigDecimal monthIncome = new BigDecimal(invest).divide(new BigDecimal(totalMonth), 2, BigDecimal.ROUND_DOWN);
		return monthIncome.doubleValue();
	}

	/**
	 * 等额本金计算获取还款方式为等额本金的总利息
	 * 
	 * @param invest
	 *            总借款额（贷款本金）
	 * @param yearRate
	 *            年利率
	 * @param month
	 *            还款总月数
	 * @return 总利息
	 */
	public static double getInterestCount(double invest, double yearRate, int totalMonth) {
		BigDecimal count = new BigDecimal(0);
		Map<Integer, Double> mapInterest = getPerMonthInterest(invest, yearRate, totalMonth);

		for (Map.Entry<Integer, Double> entry : mapInterest.entrySet()) {
			count = count.add(new BigDecimal(entry.getValue()));
		}
		return count.doubleValue();
	}

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		double invest = 10000; // 本金
		int month = 12;
		double yearRate = 0.15; // 年利率
		Map<Integer, Double> getPerMonthPrincipalInterest = getPerMonthPrincipalInterest(invest, yearRate, month);
		System.out.println("等额本金---每月本息：" + getPerMonthPrincipalInterest);
		double benjin = getPerMonthPrincipal(invest, month);
		System.out.println("等额本金---每月本金:" + benjin);
		Map<Integer, Double> mapInterest = getPerMonthInterest(invest, yearRate, month);
		System.out.println("等额本金---每月利息:" + mapInterest);

		double count = getInterestCount(invest, yearRate, month);
		System.out.println("等额本金---总利息：" + count);
	}
}
```
