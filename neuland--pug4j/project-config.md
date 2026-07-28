---
trigger: always_on
description: - Build system: Maven (pom.xml). Minimum Maven 3.0 enforced by maven-enforcer-plugin. Artifact coordinates: de.neuland-bfi:pug4j.
---

### Project-specific build and configuration

- Build system: Maven (pom.xml). Minimum Maven 3.0 enforced by maven-enforcer-plugin. Artifact coordinates: de.neuland-bfi:pug4j.
- Java version: Code is compiled for Java 8 bytecode (maven.compiler.source/target=1.8). When building with JDK ≥ 11, profile `javac-release` sets `<maven.compiler.release>8</maven.compiler.release>` to produce Java 8 compatible binaries.
- Quick build/install:
  - mvn -v  # verify Maven is available
  - mvn -T1C -DskipTests install  # parallel build, skip tests if desired
- Key plugins:
  - maven-compiler-plugin 3.13.0
  - maven-surefire-plugin 3.5.2 (JUnit 4 provider, module path disabled)
  - maven-source-plugin, maven-javadoc-plugin for publishing
- Dependencies relevant to runtime vs testing:
  - Runtime: slf4j-api 2.0.x, commons-* libraries, gson, caffeine, flexmark, optional GraalVM runtime deps (js, js-scriptengine, tools) are declared with scope runtime. You don’t need a GraalVM JDK; standard HotSpot JDK works, but Graal JS will be available at runtime via those deps when selected.

### Testing: how to configure and run

- Frameworks:
  - JUnit 4.13.2
  - Hamcrest 1.3 (assertions)
  - Snapshot testing: io.github.origin-energy java-snapshot-testing-junit4 (+ Jackson plugin) for data/regression tests. Snapshot files live under __snapshots__ directories alongside tests (e.g., src/test/java/.../__snapshots__).
  - Logging during tests: slf4j-simple (test scope) prints to console.
- Running tests:
  - All tests: mvn test
  - Single class: mvn -Dtest=Pug4JTest test
  - Single method: mvn -Dtest=Pug4JTest#testRenderDefault test
  - Repeat failed tests only: mvn -Dsurefire.rerunFailingTestsCount=1 test
- Snapshot tests:
  - Snapshot tests use Expect + SnapshotRule/ClassRule (see lexer/PugLexerTest.java). Snapshots are stored in .snap files. If behavior changes legitimately, update snapshots. The Origin Energy Snapshot Testing lib supports updating snapshots via a system property. Typical usage is:
    - mvn -Dsnapshots.update=true -Dtest=PugLexerTest test
    Consult the library docs if your environment needs a different property name/version behavior.
- Test resources/layout helpers:
  - TestFileHelper centralizes resource roots (lexer, parser, compiler, etc.) and resolves absolute paths via ClassLoader resources. Prefer using these helpers to avoid brittle paths.
  - Example: TestFileHelper.getLexerResourcePath("/cases") returns a filesystem path to src/test/resources/lexer/cases.
- Surefire configuration specifics:
  - useModulePath=false prevents JPMS/module path interference when testing on JDK 9+.
  - useFile=false prints test output to console instead of dumping to files.

### Verified example test run

- I executed a representative test locally in this repository to validate the test setup:
  - mvn -Dtest=Pug4JTest#testRenderDefault test
  - Result: BUILD SUCCESS, 1 test run, 0 failures.

### Adding a new test (recommended patterns)

- JUnit 4 style is used throughout. For basic rendering logic you can use either the static convenience API or a configured PugConfiguration. To avoid adding files when not necessary, ReaderTemplateLoader is available for in-memory templates. If your test exercises includes/extends, use FileTemplateLoader and place associated .pug files in src/test/resources.

- Minimal in-memory render test (no filesystem I/O):
  - File: src/test/java/de/neuland/pug4j/SimpleRenderTest.java
  - Contents:
    - import de.neuland.pug4j.PugConfiguration;
    - import de.neuland.pug4j.template.ReaderTemplateLoader;
    - import de.neuland.pug4j.template.PugTemplate;
    - import org.junit.Test;
    - import java.io.StringReader;
    - import java.util.HashMap;
    - import static org.junit.Assert.assertEquals;
    - public class SimpleRenderTest {
        @Test
        public void rendersInlineTemplate() throws Exception {
          String pug = "h1 Hello\np Welcome to pug4j";
          PugConfiguration config = new PugConfiguration();
          ReaderTemplateLoader loader = new ReaderTemplateLoader(new StringReader(pug), "inline");
          config.setTemplateLoader(loader);
          PugTemplate template = config.getTemplate("inline");
          String html = config.renderTemplate(template, new HashMap<>());
          assertEquals("<h1>Hello</h1><p>Welcome to pug4j</p>", html);
        }
      }
  - Run: mvn -Dtest=SimpleRenderTest test

- Filesystem-based template test (for includes/extends):
  - Place templates under src/test/resources/mycase, e.g., layout.pug and index.pug.
  - Use FileTemplateLoader with a safe base path (see Template Loader notes below).
  - Example snippet:
    - import de.neuland.pug4j.template.FileTemplateLoader;
    - Path baseDir = Paths.get("src/test/resources/mycase");
    - FileTemplateLoader loader = new FileTemplateLoader(baseDir.toString(), "pug");
    - PugConfiguration cfg = new PugConfiguration();
    - cfg.setTemplateLoader(loader);
    - PugTemplate tpl = cfg.getTemplate("index.pug");

- Parameterized/snapshot patterns:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neuland/pug4j](https://github.com/neuland/pug4j) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
