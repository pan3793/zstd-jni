# JEP 519 Compatibility Notes

## Quick Reference

**Status:** ✅ COMPATIBLE  
**Risk Level:** LOW  
**Action Required:** NONE

## Summary

This codebase is fully compatible with JEP 519 (Compact Object Headers). The extensive use of JNI is implemented using standard APIs that are unaffected by internal JVM object layout changes.

## Key Points

1. **No Unsafe Usage** - Does not use `sun.misc.Unsafe` or `jdk.internal.misc.Unsafe`
2. **Standard JNI APIs** - Uses `GetPrimitiveArrayCritical()` correctly, which is JVM-managed
3. **No Layout Assumptions** - No hardcoded object header sizes or alignment values
4. **Architecture-Agnostic** - Uses portable types (`intptr_t`) throughout

## For Developers

### JNI Memory Access Pattern

The codebase uses this safe pattern throughout:

```c
void *array_ptr = (*env)->GetPrimitiveArrayCritical(env, array, NULL);
if (array_ptr == NULL) goto error;

// Use array_ptr to access array elements
// The JVM handles offset calculations internally

(*env)->ReleasePrimitiveArrayCritical(env, array, array_ptr, JNI_ABORT);
```

**Why this is JEP 519-safe:**
- `GetPrimitiveArrayCritical()` returns a pointer to array *elements*, not the object header
- The JVM calculates element offsets: `object_address + header_size + (index * element_size)`
- When JEP 519 changes `header_size`, the JVM adjusts calculations automatically
- Native code never sees or manipulates object headers

### Testing with JEP 519

When JDK releases with JEP 519 become available:

1. Run existing test suite: `./sbt test`
2. No code changes should be needed
3. Report any issues to the maintainers

### Direct Buffer Usage

The code also uses `GetDirectBufferAddress()` for DirectByteBuffers. This is safe because:
- DirectByteBuffers use off-heap memory
- JEP 519 only affects on-heap Java objects
- No compatibility issues expected

## Further Reading

- Full analysis: [JEP-519-ANALYSIS.md](../JEP-519-ANALYSIS.md)
- JEP 519 specification: https://openjdk.org/jeps/519

---
*Last updated: 2026-01-30*
