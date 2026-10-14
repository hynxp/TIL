# Obsidian Git 동기화 점검

- 점검일: 2026-10-14
- 증상: Source Control에 변경 파일이 표시되지 않고 `Git is not ready`가 표시됨.
- 원인: macOS 기본 Git 실행 시 Xcode 약관 미동의 오류 발생.
- 조치: Git → Advanced → Custom Git binary path를 `/opt/homebrew/bin/git`으로 지정하고 Obsidian 재시작.
- 이 노트는 커밋과 원격 푸시 동작을 점검하기 위해 작성함.
