# Reviewer-Prompt

Prüfe den Phase-Commit gegen `00_START_HERE.md`, `03_ARCHITECTURE.md`, `10_PATCH_MATRIX.md` und `13_EXACT_PATCH_PLAN.md`. Kontrolliere insbesondere: Basis-SHA, keine Gewichte im Git-Diff, Scale-Scheme-Prüfung, target/draft-State, vollständige Cache-Manifeste, reproduzierbare Tests und Rollback. Akzeptiere keinen Commit mit stillen Fallbacks, unbestätigten Benchmarks oder übersprungenem Gate. Gib PASS, CONDITIONAL oder BLOCK mit konkreten Dateipfaden und Testbefehlen aus.
