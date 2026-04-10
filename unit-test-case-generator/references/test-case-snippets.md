# Test Case Snippets (6-Case Template)

这个文件提供三套可直接粘贴的测试模板：Jest、Vitest、JUnit 5。
目标：只测输入/输出与可观察行为，不绑定内部实现。

## Shared Checklist

每套模板都覆盖以下 6 类 case：

1. Normal input
2. Boundary input
3. Invalid input
4. Conflict input
5. Regression case
6. Combined constraints

---

## Jest (JavaScript/TypeScript)

```ts
import { planTimeline } from '../src/planTimeline';

describe('planTimeline', () => {
  test('normal: should return planned slots for valid input', () => {
    // Arrange
    const input = {
      events: [{ id: 'A', start: 540, end: 600 }],
      anchors: [],
      dayRange: { start: 480, end: 1320 },
    };

    // Act
    const result = planTimeline(input);

    // Assert
    expect(result.ok).toBe(true);
    expect(result.schedule.length).toBeGreaterThan(0);
  });

  test('boundary: should handle empty events', () => {
    // Arrange
    const input = {
      events: [],
      anchors: [],
      dayRange: { start: 480, end: 1320 },
    };

    // Act
    const result = planTimeline(input);

    // Assert
    expect(result.ok).toBe(true);
    expect(result.schedule).toEqual([]);
  });

  test('invalid: should throw for reversed day range', () => {
    // Arrange
    const input = {
      events: [],
      anchors: [],
      dayRange: { start: 1320, end: 480 },
    };

    // Act + Assert
    expect(() => planTimeline(input)).toThrow('dayRange');
  });

  test('conflict: should return conflict when events overlap', () => {
    // Arrange
    const input = {
      events: [
        { id: 'A', start: 540, end: 600 },
        { id: 'B', start: 580, end: 620 },
      ],
      anchors: [],
      dayRange: { start: 480, end: 1320 },
    };

    // Act
    const result = planTimeline(input);

    // Assert
    expect(result.ok).toBe(false);
    expect(result.reason).toBe('TIME_CONFLICT');
  });

  test('regression: bug-2026-03-12 should keep sorted output', () => {
    // Arrange
    const input = {
      events: [
        { id: 'B', start: 700, end: 740 },
        { id: 'A', start: 540, end: 580 },
      ],
      anchors: [],
      dayRange: { start: 480, end: 1320 },
    };

    // Act
    const result = planTimeline(input);

    // Assert
    expect(result.ok).toBe(true);
    expect(result.schedule.map((x) => x.id)).toEqual(['A', 'B']);
  });

  test('combined: should fail when anchor conflicts and range is tight', () => {
    // Arrange
    const input = {
      events: [{ id: 'A', start: 540, end: 600 }],
      anchors: [{ eventId: 'A', fixedStart: 590 }],
      dayRange: { start: 580, end: 610 },
    };

    // Act
    const result = planTimeline(input);

    // Assert
    expect(result.ok).toBe(false);
    expect(result.reason).toBe('UNSATISFIABLE_CONSTRAINTS');
  });
});
```

---

## Vitest (JavaScript/TypeScript)

```ts
import { describe, it, expect } from 'vitest';
import { planTimeline } from '../src/planTimeline';

describe('planTimeline', () => {
  it('normal: should return planned slots for valid input', () => {
    // Arrange
    const input = {
      events: [{ id: 'A', start: 540, end: 600 }],
      anchors: [],
      dayRange: { start: 480, end: 1320 },
    };

    // Act
    const result = planTimeline(input);

    // Assert
    expect(result.ok).toBe(true);
    expect(result.schedule.length).toBeGreaterThan(0);
  });

  it('boundary: should handle empty events', () => {
    // Arrange
    const input = {
      events: [],
      anchors: [],
      dayRange: { start: 480, end: 1320 },
    };

    // Act
    const result = planTimeline(input);

    // Assert
    expect(result.ok).toBe(true);
    expect(result.schedule).toEqual([]);
  });

  it('invalid: should throw for reversed day range', () => {
    // Arrange
    const input = {
      events: [],
      anchors: [],
      dayRange: { start: 1320, end: 480 },
    };

    // Act + Assert
    expect(() => planTimeline(input)).toThrowError(/dayRange/i);
  });

  it('conflict: should return conflict when events overlap', () => {
    // Arrange
    const input = {
      events: [
        { id: 'A', start: 540, end: 600 },
        { id: 'B', start: 580, end: 620 },
      ],
      anchors: [],
      dayRange: { start: 480, end: 1320 },
    };

    // Act
    const result = planTimeline(input);

    // Assert
    expect(result.ok).toBe(false);
    expect(result.reason).toBe('TIME_CONFLICT');
  });

  it('regression: bug-2026-03-12 should keep sorted output', () => {
    // Arrange
    const input = {
      events: [
        { id: 'B', start: 700, end: 740 },
        { id: 'A', start: 540, end: 580 },
      ],
      anchors: [],
      dayRange: { start: 480, end: 1320 },
    };

    // Act
    const result = planTimeline(input);

    // Assert
    expect(result.ok).toBe(true);
    expect(result.schedule.map((x) => x.id)).toEqual(['A', 'B']);
  });

  it('combined: should fail when anchor conflicts and range is tight', () => {
    // Arrange
    const input = {
      events: [{ id: 'A', start: 540, end: 600 }],
      anchors: [{ eventId: 'A', fixedStart: 590 }],
      dayRange: { start: 580, end: 610 },
    };

    // Act
    const result = planTimeline(input);

    // Assert
    expect(result.ok).toBe(false);
    expect(result.reason).toBe('UNSATISFIABLE_CONSTRAINTS');
  });
});
```

---

## JUnit 5 (Java)

```java
import static org.junit.jupiter.api.Assertions.*;
import org.junit.jupiter.api.Test;

class TimelinePlannerTest {

    @Test
    void normal_shouldReturnPlannedSlotsForValidInput() {
        // Arrange
        PlanInput input = new PlanInput(
            List.of(new Event("A", 540, 600)),
            List.of(),
            new DayRange(480, 1320)
        );

        // Act
        PlanResult result = TimelinePlanner.plan(input);

        // Assert
        assertTrue(result.ok());
        assertFalse(result.schedule().isEmpty());
    }

    @Test
    void boundary_shouldHandleEmptyEvents() {
        // Arrange
        PlanInput input = new PlanInput(
            List.of(),
            List.of(),
            new DayRange(480, 1320)
        );

        // Act
        PlanResult result = TimelinePlanner.plan(input);

        // Assert
        assertTrue(result.ok());
        assertEquals(0, result.schedule().size());
    }

    @Test
    void invalid_shouldThrowForReversedDayRange() {
        // Arrange
        PlanInput input = new PlanInput(
            List.of(),
            List.of(),
            new DayRange(1320, 480)
        );

        // Act + Assert
        IllegalArgumentException ex = assertThrows(
            IllegalArgumentException.class,
            () -> TimelinePlanner.plan(input)
        );
        assertTrue(ex.getMessage().contains("dayRange"));
    }

    @Test
    void conflict_shouldReturnConflictWhenEventsOverlap() {
        // Arrange
        PlanInput input = new PlanInput(
            List.of(new Event("A", 540, 600), new Event("B", 580, 620)),
            List.of(),
            new DayRange(480, 1320)
        );

        // Act
        PlanResult result = TimelinePlanner.plan(input);

        // Assert
        assertFalse(result.ok());
        assertEquals("TIME_CONFLICT", result.reason());
    }

    @Test
    void regression_bug20260312_shouldKeepSortedOutput() {
        // Arrange
        PlanInput input = new PlanInput(
            List.of(new Event("B", 700, 740), new Event("A", 540, 580)),
            List.of(),
            new DayRange(480, 1320)
        );

        // Act
        PlanResult result = TimelinePlanner.plan(input);

        // Assert
        assertTrue(result.ok());
        assertEquals(List.of("A", "B"), result.scheduleIds());
    }

    @Test
    void combined_shouldFailWhenAnchorConflictsAndRangeIsTight() {
        // Arrange
        PlanInput input = new PlanInput(
            List.of(new Event("A", 540, 600)),
            List.of(new Anchor("A", 590)),
            new DayRange(580, 610)
        );

        // Act
        PlanResult result = TimelinePlanner.plan(input);

        // Assert
        assertFalse(result.ok());
        assertEquals("UNSATISFIABLE_CONSTRAINTS", result.reason());
    }
}
```

## Usage Notes

- 先替换示例中的函数名、输入结构、错误码、断言目标。
- 保留 case 分类与 AAA 结构。
- 回归用例必须绑定真实缺陷编号或时间点，避免“伪回归”。
- 不要新增内部调用顺序断言，除非该顺序本身是业务契约。
