# Tables this stage reads or writes

Header rows only, read from the live tables on 2026-10-06. No data rows are in any repo.

## `data/postings.csv`

32038 rows. Columns:

`posting_id`, `account_id`, `mirror_account_id`, `mirror_site_id`, `mirror_basis`, `site_id`, `site_resolution`, `site_candidates`, `city_slug`, `city_raw`, `also_at`, `icp_fit`, `role_title`, `role_title_source`, `role_slug`, `req_id`, `url_form`, `careers_host`, `job_family`, `job_family_rule`, `seniority_rank`, `seniority_rule`, `excluded`, `tier2_admit`, `tier2_score`, `url`, `first_seen`, `first_seen_kind`, `last_seen`, `last_seen_kind`, `seen_count`, `period_first`, `period_last`, `posted_date`, `posted_date_source`, `extraction_tier`, `body_chars`, `extract_status`, `captured_at`, `updated_at`, `last_run_id`

## `data/posting_extracts.csv`

32 rows. Columns:

`posting_id`, `account_id`, `site_id_at_extract`, `team_name`, `team_slug`, `team_acronym`, `team_name_conf`, `parent_org`, `parent_slug`, `parent_conf`, `supports_teams`, `supports_slugs`, `reports_to_title`, `hiring_manager`, `hiring_manager_conf`, `site_stated`, `posted_date_verbatim`, `tech_stack`, `tech_stack_raw`, `vocabulary`, `autonomy_flag`, `acquisition_flag`, `n_quotes`, `n_quotes_verified`, `n_claims_dropped`, `body_chars`, `extraction_tier`, `extract_run_id`, `extract_status`, `extract_notes`, `captured_at`

## `data/posting_quotes.csv`

403 rows. Columns:

`quote_id`, `posting_id`, `account_id`, `claim_type`, `subject`, `subject_slug`, `quote_text`, `verbatim_verified`, `match_method`, `display_date`, `date_kind`, `extraction_tier`, `extract_run_id`, `captured_at`

## `data/org_units.csv`

89 rows. Columns:

`unit_id`, `account_id`, `unit_slug`, `unit_name_display`, `unit_name_generated`, `acronym`, `acronym_source`, `aliases`, `unit_kind`, `site_id`, `site_resolution`, `site_candidates`, `parent_unit_id`, `parent_label`, `parent_conf`, `n_postings`, `n_postings_tier3plus`, `n_quotes`, `first_evidence_date`, `last_evidence_date`, `evidence_date_kind`, `job_families`, `seniority_max`, `tech_stack`, `vocabulary`, `hiring_managers`, `evidence_posting_ids`, `evidence_quote_ids`, `status`, `confidence`, `confidence_basis`, `gap_flags`, `human_state`, `human_note`, `generated_at`

## `data/org_edges.csv`

98 rows. Columns:

`edge_id`, `account_id`, `edge_type`, `src_unit_id`, `dst_unit_id`, `dst_site_id`, `src_label`, `dst_label`, `n_postings`, `n_postings_tier3plus`, `n_quotes`, `n_independent_postings`, `first_evidence_date`, `last_evidence_date`, `evidence_date_kind`, `evidence_posting_ids`, `evidence_quote_ids`, `confidence`, `confidence_basis`, `status`, `human_state`, `human_note`, `generated_at`

## `data/org_site_coverage.csv`

604 rows. Columns:

`account_id`, `site_id`, `site_name`, `city`, `icp_fit`, `n_indexed`, `n_admitted`, `n_bodies`, `n_extracts`, `n_units`, `first_seen`, `last_seen`, `coverage_state`, `scanned_at`

## `data/posting_rollup.csv`

640 rows. Columns:

`rollup_id`, `scope`, `account_id`, `site_id`, `period`, `dim_kind`, `dim_key`, `n`, `denom`, `share`, `account_alltime_share`, `self_index`, `peer_median_share`, `peer_n_accounts`, `peer_index`, `bias_flag`, `bias_note`, `generated_at`

## `data/careers.csv`

49 rows. Columns:

`account_id`, `company`, `careers_host`, `pattern_kind`, `req_path_glob`, `url_sample`, `req_urls_found`, `city_from_url`, `probe_status`, `candidates_tried`, `probed_at`, `notes`, `workday_host`, `workday_site`

## `data/careers_probe.csv`

40 rows. Columns:

`account_id`, `company`, `careers_host`, `robots`, `robots_allows_jobs`, `sitemap_url`, `sitemap_job_urls`, `jsonld_sampled`, `jsonld_hits`, `date_posted_sample`, `location_structured`, `req_identifier`, `body_chars_median`, `verdict`, `notes`, `probed_at`

## `data/ats_census.csv`

49 rows. Columns:

`account_id`, `company`, `careers_url`, `ats`, `ats_host`, `bucket`, `api_shape`, `jsonld_jobposting`, `http`, `notes`, `probed_at`

## `data/quarantine_org.csv`

0 rows. Columns:

`posting_id`, `account_id`, `claim_type`, `subject`, `quote_text`, `reason`, `body_chars`, `checked_at`

## `data/jobs_runs.csv`

14 rows. Columns:

`run_id`, `kind`, `started_at`, `finished_at`, `accounts`, `urls_seen`, `new_urls`, `extracted`, `quarantined`, `verify_pass_rate`, `notes`

## `data/unmatched_cities.csv`

1590 rows. Columns:

`account_id`, `city_raw`, `key`, `n`, `n_admitted`, `first_seen`, `last_seen`, `verdict`
