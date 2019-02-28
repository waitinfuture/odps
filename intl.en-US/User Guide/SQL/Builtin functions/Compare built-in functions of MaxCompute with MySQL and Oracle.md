# Compare built-in functions of MaxCompute with MySQL and Oracle {#concept_fbn_zcn_sfb .concept}

The following table compares the built-in functions used in MaxCompute with those of MySQL and Oracle.

|Function type|MaxCompute|MySQL|Oracle|
|[Date functions](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#)|[DATEDIFF](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_xl2_nsl_vdb)|DATEDIFF|MONTHS\_BETWEEN|
|[DATEADD](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_qjz_lrl_vdb)|DATE\_ADD|N/A|
|[DATEPART](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_am4_xtl_vdb)|DATE\_FORMAT|EXTRACT \(datetime\)|
|[DATETRUNC](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_zbr_d5l_vdb)|DATE\_FORMAT|EXTRACT \(datetime\)|
|[FROM\_UNIXTIME](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_yzv_j5l_vdb)|FROM\_UNIXTIME|N/A|
|[GETDATE](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_o4p_45l_vdb)|NOW|CURRENT\_DATE|
|[ISDATE](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_rzl_s5l_vdb)|STR\_TO\_DATE \(If 'false' is returned, the string cannot be converted to date.\)|N/A|
|[LASTDAY](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_vhk_w2m_vdb)|LAST\_DAY|LAST\_DAY|
|[TO\_DATE](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_b3z_1fm_vdb)|STR\_TO\_DATE\(\)|DATE|
|[TO\_CHAR](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_a2d_rfm_vdb)|DATE\_FORMAT|TO\_CHAR \(datetime\)|
|[UNIX\_TIMESTAMP](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_k4r_zfm_vdb)|UNIX\_TIMESTAMP|N/A|
|[WEEKDAY](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_g41_2gm_vdb)|WEEKDAY|N/A|
|[WEEKOFYEAR](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_rjv_hgm_vdb)|WEEKOFYEAR|N/A|
|[YEAR](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_gb4_g3m_vdb)|YEAR|YEAR|
|[QUARTER](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_ckz_m3m_vdb)|QUARTER|QUARTER|
|[MONTH](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_atw_s3m_vdb)|MONTH|MONTH|
|[DAY](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_lt5_w3m_vdb)|DAY|DAY|
|[DAYOFMONTH](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_k4p_fjm_vdb)|DAYOFMONTH|N/A|
|[HOUR](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_lrl_jjm_vdb)|HOUR|HOUR|
|[MINUTE](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_o15_mjm_vdb)|MINUTE|MINUTE|
|[CURRENT\_TIMESTAMP](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_r1r_sjm_vdb)|CURRENT\_TIMESTAMP|CURRENT\_TIMESTAMP|
|[ADD\_MONTHS](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_psn_vjm_vdb)|ADDDATE|ADD\_MONTHS|
|[LAST\_DAY](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_idn_1km_vdb)|LAST\_DAY|N/A|
|[NEXT\_DAY](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_eqf_2km_vdb)|N/|NEXT\_DAY|
|[MONTHS\_BETWEEN](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#section_cbq_4km_vdb)|timestampdiff|MONTHS\_BETWEEN|
|[Mathematical functions](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#)|[ABS](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#)|ABS|ABS|
|[ACOS](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_cfp_qmm_vdb)|ACOS|ACOS|
|[ASIN](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#)|ASIN|ASIN|
|[ATAN](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_odw_jnm_vdb)|ATAN|ATAN|
|[CEIL](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_ugm_k4m_vdb)|CEIL|CEIL|
|[CONV](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_tkx_q4m_vdb)|CONV|N/A|
|[COS](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_tpy_z4m_vdb)|COS|COS|
|[COSH](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_tnp_gpm_vdb)|N/A|COSH|
|[COT](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_hhz_lpm_vdb)|COT|COT|
|[EXP](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_q1n_rpm_vdb)|EXP|EXP|
|[FLOOR](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_yrw_wpm_vdb)|FLOOR|FLOOR|
|[LN](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_pdm_fqm_vdb)|LN|LN|
|[LOG](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_iwc_4qm_vdb)|LOG|LOG|
|[POW](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_gmv_wqm_vdb)|POW|POWER|
|[RAND](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_qlv_2rm_vdb)|RAND|N/A|
|[ROUND](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_ocf_jrm_vdb)|ROUND|ROUND|
|[SIN](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_tky_gvm_vdb)|SIN|SIN|
|[SINH](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_ccf_gym_vdb)|N/A|SINH|
|[SQRT](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_nns_lym_vdb)|SQRT|SQRT|
|[TAN](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_ibd_rym_vdb)|TAN|TAN|
|[TANH](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_pfh_wym_vdb)|N/A|TANH|
|[TRUNC](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_yly_1zm_vdb)|TRUNCATE|TRUNC|
|[LOG2](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_dh3_tzm_vdb)|LOG2|LOG|
|[LOG10](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_bjc_zzm_vdb)|LOG10|LOG|
|[BIN](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_ucn_21n_vdb)|BIN|BITAND|
|[HEX](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_nxv_j1n_vdb)|HEX|RAWTOHEX|
|[UNHEX](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_k5x_51n_vdb)|UNHEX|HEXTORAW|
|[RADIANS](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_dwg_1bn_vdb)|RADIANS|RADIANS|
|[DEGREES](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_adl_hbn_vdb)|DEGREES|DEGREES|
|[SIGN](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_gq5_lbn_vdb)|SIGN|SIGN|
|[E](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_yfc_zbn_vdb)|N/A|EXP|
|[PI](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_hhc_dcn_vdb)|PI|PI|
|[FACTORIAL](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_umk_gcn_vdb)|N/A|N/A|
|[CBRT](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_frl_lcn_vdb)|N/A|N/A|
|[SHIFTLEFT](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_k4z_pcn_vdb)|<<|N/A|
|[SHIFTRIGHT](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_iyl_vcn_vdb)|\>\>|N/A|
|[SHIFTRIGHTUNSIGNED](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#section_h2f_1dn_vdb)|\>\>\>|N/A|
|[Window functions](reseller.en-US/User Guide/SQL/Builtin functions/Window functions.md#)|[DENSE\_RANK](reseller.en-US/User Guide/SQL/Builtin functions/Window functions.md#)|DENSE\_RANK|DENSE\_RANK|
|[RANK](reseller.en-US/User Guide/SQL/Builtin functions/Window functions.md#section_yvx_jb1_wdb)|RANK|RANK|
|[LAG](reseller.en-US/User Guide/SQL/Builtin functions/Window functions.md#section_dbf_xb1_wdb)|LAG|LAG|
|[LEAD](reseller.en-US/User Guide/SQL/Builtin functions/Window functions.md#section_s5f_jc1_wdb)|LEAD|LEAD|
|[PERCENT\_RANK](reseller.en-US/User Guide/SQL/Builtin functions/Window functions.md#section_lmk_tc1_wdb)|PERCENT\_RANK|PERCENT\_RANK|
|[ROW\_NUMBER](reseller.en-US/User Guide/SQL/Builtin functions/Window functions.md#section_cm1_cd1_wdb)|ROW\_NUMBER|ROW\_NUMBER|
|[CLUSTER\_SAMPLE](reseller.en-US/User Guide/SQL/Builtin functions/Window functions.md#section_mst_md1_wdb)|N/A|N/A|
|[NTILE](reseller.en-US/User Guide/SQL/Builtin functions/Window functions.md#section_gjj_c21_wdb)|NTILE|NTILE|
|[Aggregate functions](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#)|[COUNT](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#section_x1k_xq1_wdb)|COUNT|COUNT|
|[AVG](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#section_oxf_mr1_wdb)|AVG|AVG|
|[MAX](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#section_rys_tr1_wdb)|MAX|MAX|
|[MIN](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#section_mll_yr1_wdb)|MIN|MIN|
|[MEDIAN](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#section_m5y_cs1_wdb)|N/A|MEDIAN|
|[STDDEV](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#section_gg5_dv1_wdb)|STDDEV|STDDEV|
|[STDDEV\_SAMP](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#section_sgk_jv1_wdb)|STDDEV\_SAMP|STDDEV\_SAMP|
|[SUM](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#section_okf_4v1_wdb)|SUM|SUM|
|[WM\_CONCAT](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#section_ddm_tv1_wdb)|GROUP\_CONCAT|WM\_CONCAT|
|[COLLECT\_LIST](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#section_fth_1w1_wdb)|N/A|COLLECT|
|[COLLECT\_SET](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#section_skl_fw1_wdb)|N/A|COLLECT|
|[VARIANCE/VAR\_POP](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#section_emm_lw1_wdb)|VAR\_POP|VARIANCE/VAR\_POP|
|[VAR\_SAMP](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#section_dt5_tw1_wdb)|VAR\_SAMP|VAR\_SAMP|
|[COVAR\_POP](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#section_hng_1x1_wdb)|N/A|COVAR\_POP|
|[COVAR\_SAMP](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#section_zvm_gx1_wdb)|N/A|COVAR\_SAMP|
|[PERCENTILE](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#section_lqv_lx1_wdb)|N/A|N/A|
|[String functions](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#)|[CHAR\_MATCHCOUNT](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_mnd_gvz_vdb)|N/A|N/A|
|[CHR](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_s5r_lwz_vdb)|CHAR|CHR|
|[CONCAT](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_xxj_wwz_vdb)|CONCAT|CONCAT|
|[GET\_JSON\_OBJECT](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_cdt_gxz_vdb)|JSON\_EXTRACT\(\)|N/A|
|[INSTR](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_vft_yxz_vdb)|INSTR|INSTR|
|[IS\_ENCODING](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_qdj_kyz_vdb)|N/A|N/A|
|[KEYVALUE](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_lnq_tyz_vdb)|N/A|N/A|
|[LENGTH](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_ewt_jzz_vdb)|LENGTH|LENGTH|
|[LENGTHB](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_o3y_pzz_vdb)|LENGTHB|LENGTHB|
|[MD5](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_hbw_xzz_vdb)|MD5|N/A|
|[REGEXP\_EXTRACT](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_ms1_lc1_wdb)|N/A|N/A|
|[REGEXP\_INSTR](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_jpn_5c1_wdb)|REGEXP\_INSTR|REGEXP\_INSTR|
|[REGEXP\_REPLACE](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_k2w_2d1_wdb)|REGEXP\_REPLACE|REGEXP\_REPLACE|
|[REGEXP\_SUBSTR](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_k5b_qd1_wdb)|REGEXP\_SUBSTR|REGEXP\_SUBSTR|
|[REGEXP\_COUNT](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_hxs_yd1_wdb)|N/A|REGEXP\_COUNT|
|[SPLIT\_PART](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_ecy_k21_wdb)|N/A|N/A|
|[SUBSTR](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_nkj_1f1_wdb)|SUBSTR|SUBSTR|
|[SUBSTRING](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_s1h_3f1_wdb)|SUBSTRING|SUBSTR|
|[TOLOWER](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_nzz_lg1_wdb)|LOWER|LOWER|
|[TOUPPER](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_qvg_sg1_wdb)|UPPER|UPPER|
|[TO\_CHAR](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_lgq_wg1_wdb)|DATE\_FORMAT|TO\_CHAR|
|[TRIM](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_mf1_3h1_wdb)|TRIM|TRIM|
|[LTRIM](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_shc_d21_wdb)|LTRIM|LTRIM|
|[RTRIM](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_gtk_rh1_wdb)|RTRIM|LTRIM|
|[REVERSE](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_or4_d31_wdb)|REVERSE|REVERSE|
|[SPACE](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_a25_h31_wdb)|SPACE|SPACE|
|[REPEAT](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_gnq_m31_wdb)|REPEAT|REPEAT|
|[ASCII](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_hgm_s31_wdb)|ASCII|ASCII|
|[CONCAT\_WS](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_xnf_sj1_wdb)|CONCAT\_WS|N/A|
|[LPAD](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_mcj_zj1_wdb)|LPAD|LPAD|
|[RPAD](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_k1f_3k1_wdb)|RPAD|RPAD|
|[REPLACE](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_ln3_5k1_wdb)|REPLACE|REPLACE|
|[SOUNDEX](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_z2z_1l1_wdb)|SOUNDEX|SOUNDEX|
|[SUBSTRING\_INDEX](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_uw3_hl1_wdb)|SUBSTRING\_INDEX|N/A|
|[TRANSLATE](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_bk1_nl1_wdb)|N/A|TRANSLATE|
|[URL\_DECODE](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_kwp_lbp_32b)|N/A|N/A|
|[URL\_ENCODE](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_hy4_ccp_32b)|N/A|PERCENTILE\_CONT|
|CRC32|CRC32|N/A|
|[Other functions](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#)|[CAST](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_bpc_dy1_wdb)|CAST|CAST|
|[COALESCE](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_dts_3y1_wdb)|COALESCE|COALESCE|
|[DECODE](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_ygq_4y1_wdb)|N/A|DECODE|
|[GET\_IDCARD\_AGE](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_j2q_1z1_wdb)|N/A|N/A|
|[GET\_IDCARD\_BIRTHDAY](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_tfq_dz1_wdb)|N/A|N/A|
|[GET\_IDCARD\_SEX](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_akt_gz1_wdb)|N/A|N/A|
|[GREATEST](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_n1g_kz1_wdb)|GREATEST|N/A|
|[ORDINAL](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_pcj_pz1_wdb)|N/A|GREATEST|
|[LEAST](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_m1b_xz1_wdb)|LEAST|N/A|
|[MAX\_PT](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_yyf_d1b_wdb)|N/A|LEAST|
|[UUID](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_gtg_j1b_wdb)|UUID|N/A|
|[SAMPLE](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_vvl_l1b_wdb)|N/A|UID|
|[IF](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_mg5_1bb_wdb)|IF|IF|
|[CASE WHEN](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_axm_v1b_wdb)|CASE WHEN|CASE WHEN|
|[SPLIT](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_omq_nbb_wdb)|SPLIT|N/A|
|[STR\_TO\_MAP](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_p1z_xrj_dfb)|N/A|N/A|
|[EXPLODE](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_wgw_xbb_wdb)|N/A|N/A|
|[MAP](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_bzn_hcb_wdb)|N/A|N/A|
|[MAP\_KEYS](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_mym_clb_wdb)|N/A|N/A|
|[MAP\_VALUES](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_pd3_jlb_wdb)|N/A|N/A|
|[NVL](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_c2j_hbb_wdb)|IFNULL|N/A|
|[ARRAY](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_zcz_4lb_wdb)|N/A|N/A|
|[SIZE](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_nr5_vlb_wdb)|N/A|N/A|
|[ARRAY\_CONTAINS](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_rk2_2mb_wdb)|N/A|N/A|
|[POSEXPLODE](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_ilc_xmb_wdb)|N/A|N/A|
|[TRANS\_ARRAY](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_y21_vnb_wdb)|N/A|N/A|
|[INLINE](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_w3l_pnb_wdb)|N/A|N/A|
|[NAMED\_STRUCT](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_ecs_3nb_wdb)|N/A|N/A|

