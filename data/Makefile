all: sex_by_age.json college_pop.csv mit_enrollment.csv mit_enrollment.json

sex_by_age.json:
	curl 'http://api.censusreporter.org/1.0/data/show/latest?table_ids=B01001&geo_ids=04000US01,04000US02,04000US04,04000US05,04000US06,04000US08,04000US09,04000US10,04000US11,04000US12,04000US13,04000US15,04000US16,04000US17,04000US18,04000US19,04000US20,04000US21,04000US22,04000US23,04000US24,04000US25,04000US26,04000US27,04000US28,04000US29,04000US30,04000US31,04000US32,04000US33,04000US34,04000US35,04000US36,04000US37,04000US38,04000US39,04000US40,04000US41,04000US42,04000US44,04000US45,04000US46,04000US47,04000US48,04000US49,04000US50,04000US51,04000US53,04000US54,04000US55,04000US56' | jq '.' > sex_by_age.json

college_pop.csv: sex_by_age.json
	cat sex_by_age.json | jq -c '.data | .[] | ."B01001"' -S | json2csv -p -k estimate.B01001007,error.B01001007,estimate.B01001008,error.B01001008,estimate.B01001009,error.B01001009,estimate.B01001031,error.B01001031,estimate.B01001032,error.B01001032,estimate.B01001033,error.B01001033 | csvjoin states.csv - > college_pop.csv

mit_enrollment.csv: college_pop.csv mit_usug_fa14.csv
	csvcut college_pop.csv -c estimate.B01001007,estimate.B01001008,estimate.B01001009,estimate.B01001031,estimate.B01001032,estimate.B01001033 | tail -n +2 | sed -e 's/,/+/g' | bc > est.csv
	(echo 'estimate'; cat est.csv) > estimates.csv && rm est.csv
	csvcut college_pop.csv -c error.B01001007,error.B01001008,error.B01001009,error.B01001031,error.B01001032,error.B01001033 | tail -n +2 | sed -e 's/,/+/g' | bc > err.csv
	(echo 'error'; cat err.csv) > errors.csv && rm err.csv
	csvjoin college_pop.csv estimates.csv errors.csv | csvjoin - mit_usug_fa14.csv -c "name,location" | csvcut -C estimate.B01001007,error.B01001007,estimate.B01001008,error.B01001008,estimate.B01001009,error.B01001009,estimate.B01001031,error.B01001031,estimate.B01001032,error.B01001032,estimate.B01001033,error.B01001033,location > enr.csv
	csvcut enr.csv -c enrollment | tail -n +2 | awk '{ total += $$1 } END { while (count++<51) print total }' > sum.csv
	(echo 'total'; cat sum.csv) > total.csv && rm sum.csv
	csvcut enr.csv -c enrollment | csvjoin - total.csv | tail -n +2 | sed -e 's/,/\//g' -e 's/^/scale=6;/' | bc > pct.csv
	(echo 'percent'; cat pct.csv) > percentages.csv && rm pct.csv
	csvjoin enr.csv percentages.csv | sed -e 's/^04000US//' > mit_enrollment.csv
	rm estimates.csv errors.csv enr.csv total.csv percentages.csv

mit_enrollment.json: mit_enrollment.csv
	csvjson mit_enrollment.csv -k geoid -i 2 | jq -c '.[] | {(.geoid): [.name, .estimate, .error, .enrollment, .percent]}' | sed -E -e 's/\}/,/g' -e 's/\{//g' -e '1s/^/\{/' -e '51s/.$$/\}/' > mit_enrollment.json