요즘 유행하는 LLM wiki 에서 생성된 문서 중에 DBMS 관련 문서만 여기 repo 에 자동으로 업로드한다.

# 문서 생성 절차
1. raw 문서 생성 ( *.txt , web clipping )
2. LLM (claude code)로 markdown 문서로 가공
3. 가공된 문서는  category 별로  지정 폴더로 자동 분류된다.
4. github mcp 를 통해서 연관 category 폴더에  문서  자동 upload

2,3,4 단계는  claude skill command 를 호출하는 python 에서 수행한다.

# local에서 문서 참고
* pc(local) 에서는  옵시디언이나 visual code 를 viewer로 해서 문서를 확인/편집한다.

# Outside home 에서 문서참고
* github 에 upload 된  문서를 참고한다.
* obsidian sync 는 유료이다.

# 문서 history 확인
* root 폴더의  DOC_HISTORY.md 를 확인한다.

