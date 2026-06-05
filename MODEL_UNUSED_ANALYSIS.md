# Model 部分未使用内容与方案合理性分析

## 1. Final model 实际使用了哪些内容

当前最终模型为 **Logistic Regression + `core + complaint text`**。该方案使用的特征组包括：

| Feature group | 使用内容 | 说明 |
|---|---|---|
| `core_demographics` | `age`, `age_group`, `sex`, `weight_kg`, `height_cm`, `bmi` | 基础人口统计和体型信息 |
| `core_vitals` | `systolic_bp`, `diastolic_bp`, `heart_rate`, `respiratory_rate`, `temperature_c`, `spo2`, `gcs_total`, `mental_status_triage`, `pain_score_clean` | 分诊时可直接获得的核心临床信息 |
| `complaint_structured` | `chief_complaint_system`, `pain_location`, `chief_complaint_char_len`, `chief_complaint_word_count`, `chief_complaint_missing` | 主诉的结构化和简单统计特征 |
| `complaint_text` | `chief_complaint_raw_clean` | 清洗后的主诉文本，经 TF-IDF 转换 |

因此，最终模型在文本向量化之前使用了 **21 个输入特征**。文本部分使用 TF-IDF，参数为 `min_df=3`, `max_features=5000`, `ngram_range=(1,2)`, `sublinear_tf=True`。Logistic Regression 参数为 `solver=liblinear`, `class_weight=balanced`, `C=1.0`, `max_iter=500`, `random_state=42`。

## 2. EDA 和 preprocessing 中没有进入最终模型的内容

| 未进入最终模型的内容 | 来源 | 没有使用的原因 |
|---|---|---|
| `patient_id` | 原始数据 / preprocessing 保留 | 只用于合并、行对齐和生成 submission，不能作为预测特征，否则可能造成 ID shortcut learning。 |
| `disposition`, `ed_los_hours` | EDA 中用于回顾性分析 | 这两个变量是分诊后的 outcome，属于明显 leakage，不能用于模型训练。 |
| `site_id`, `triage_nurse_id` | workflow / operational identifiers | 最终模型主动避免使用，因为它们可能编码医院站点、护士习惯或流程差异，而不是患者本身的临床状态。 |
| workflow context | `arrival_mode`, `arrival_hour`, `arrival_day`, `arrival_month`, `arrival_season`, `shift`, `transport_origin`, `language`, `insurance_type` | 这些特征在 full model 和 ablation 中被测试，但没有进入最终模型。原因是最终方案优先选择更小、更临床相关、泛化风险更低的特征集。 |
| derived clinical features | `mean_arterial_pressure`, `pulse_pressure`, `shock_index`, `news2_score`, `high_news2`, `low_gcs`, `very_low_gcs`, `low_spo2`, `very_low_spo2`, `tachypnea`, `fever`, `severe_pain`, `hypotension`, `altered_mental_status_flag` | 这些工程特征用于比较 derived clinical signal 的提升，但最终模型没有依赖它们。核心生命体征和主诉文本已经达到很高的 grouped validation 表现，因此更复杂的规则特征没有带来必要收益。 |
| patient history features | 所有 `hx_*`, `comorbidity_count`, `high_risk_history_flag`, prior visit/admission/medication/comorbidity counts | 病史特征在 ablation 中被评估，但没有进入最终模型。它们可能有临床价值，但当前验证结果显示主诉文本和核心临床特征已经足够强。 |
| complaint red-flag indicators | `complaint_redflag_*` | 这些关键词规则提升了可解释性，但最终模型直接使用 cleaned complaint text 的 TF-IDF 表达，已经包含了更细粒度的主诉信息。 |
| quality flags | `*_missing`, `*_invalid`, `missing_vital_count`, `bp_inconsistent` | preprocessing 中创建了缺失和异常值指示器，但最终模型没有使用。最终模型只通过 imputation 处理缺失值，没有显式建模数据质量信号。 |
| raw text / raw pain score | `chief_complaint_raw`, `pain_score` | `chief_complaint_raw` 清洗后才进入 TF-IDF；`pain_score` 清洗为 `pain_score_clean` 后才使用，避免无效值影响训练。 |