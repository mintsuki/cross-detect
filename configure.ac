AC_INIT(cross-detect, 0.0, [https://github.com/mintsuki/cross-detect/issues], [cross-detect])

AC_PREREQ(2.69)

if test "x$TRIPLET" = "x"; then
    AC_MSG_ERROR([error: must specify TRIPLET])
fi

AC_PROG_SED

ARCHITECTURE="$(echo "$TRIPLET" | $SED 's/-.*//g')"

if test -f "$ARCHITECTURE-toolchain.mk"; then
    exit 0
fi

AC_PROG_GREP

ENFORCE_TOOLCHAIN=no
if test "x$CROSS_TOOLCHAIN" = "x"; then
    CROSS_TOOLCHAIN="$TRIPLET"
else
    ENFORCE_TOOLCHAIN=yes
fi

AC_DEFUN([CC_ERROR_MSG], [
    AC_MSG_ERROR([no suitable CROSS_CC found])
])

if ! test "x$CROSS_CC" = "x"; then
    AC_CHECK_PROG([CROSS_CC_0], [$CROSS_CC], [yes])
    if ! test "x$CROSS_CC_0" = "xyes"; then
        AC_MSG_ERROR([CROSS_CC ($CROSS_CC) not found])
    fi
else
    if test "$CROSS_TOOLCHAIN" = 'llvm'; then
        CROSS_CC='clang'
    else
        CROSS_CC="$CROSS_TOOLCHAIN-gcc"
    fi
    AC_CHECK_PROG([CROSS_CC_1], [$CROSS_CC], [yes])
    if ! test "x$CROSS_CC_1" = "xyes"; then
        if test "x$ENFORCE_TOOLCHAIN" = "xyes"; then
            CC_ERROR_MSG
        fi
        CROSS_CC="clang"
        AC_CHECK_PROG([CROSS_CC_2], [$CROSS_CC], [yes])
        if ! test "x$CROSS_CC_2" = "xyes"; then
            CROSS_CC="gcc"
            AC_CHECK_PROG([CROSS_CC_3], [$CROSS_CC], [yes])
            if ! test "x$CROSS_CC_3" = "xyes"; then
                CROSS_CC="cc"
                AC_CHECK_PROG([CROSS_CC_4], [$CROSS_CC], [yes])
                if ! test "x$CROSS_CC_4" = "xyes"; then
                    CC_ERROR_MSG
                fi
            fi
        fi
    fi
fi
AC_SUBST(CROSS_CC, $CROSS_CC)

if ! $CROSS_CC --version >/dev/null 2>&1; then
    CC_ERROR_MSG
fi

if $CROSS_CC --version | $GREP clang >/dev/null 2>&1; then
    CROSS_CC="$CROSS_CC --target=$TRIPLET"
elif $CROSS_CC --version | $GREP 'Free Software Foundation' >/dev/null 2>&1; then
    true
else
    CC_ERROR_MSG
fi

if ! $CROSS_CC -dumpmachine >/dev/null 2>&1; then
    CC_ERROR_MSG
fi

CC_MACHINE="$($CROSS_CC -dumpmachine | $SED 's/-.*//g')"

if test "x$CC_MACHINE" = "xamd64"; then
    CC_MACHINE=x86_64
fi

if test "x$CC_MACHINE" = "xx86_64"; then
    case "$ARCHITECTURE" in
        i?86)
            CC_MACHINE="$ARCHITECTURE"
            ;;
        *)
            ;;
    esac
fi

if ! test "x$CC_MACHINE" = "x$ARCHITECTURE"; then
    CC_ERROR_MSG
fi

AC_DEFUN([GET_BINUTILS_PROG], [
    if ! test "x$CROSS_$1" = "x"; then
        AC_CHECK_PROG([CROSS_$1_0], [$CROSS_$1], [yes])
        if ! test "x$CROSS_$1_0" = "xyes"; then
            AC_MSG_ERROR([CROSS_$1 ($CROSS_$1) not found])
        fi
    else
        if test "$CROSS_TOOLCHAIN" = 'llvm'; then
            case '$2' in
                ld) CROSS_$1='ld.lld' ;;
                'c++filt') CROSS_$1='llvm-cxxfilt' ;;
                *) CROSS_$1='llvm-$2' ;;
            esac
        else
            CROSS_$1="$CROSS_TOOLCHAIN-$2"
        fi
        AC_CHECK_PROG([CROSS_$1_1], [$CROSS_$1], [yes])
        if ! test "x$CROSS_$1_1" = "xyes"; then
            if test "x$ENFORCE_TOOLCHAIN" = "xyes"; then
                AC_MSG_ERROR([CROSS_$1 ($CROSS_$1) not found])
            fi
            case '$2' in
                ld) CROSS_$1='ld.lld' ;;
                'c++filt') CROSS_$1='llvm-cxxfilt' ;;
                *) CROSS_$1='llvm-$2' ;;
            esac
            AC_CHECK_PROG([CROSS_$1_2], [$CROSS_$1], [yes])
            if ! test "x$CROSS_$1_2" = "xyes"; then
                CROSS_$1='$2'
                AC_CHECK_PROG([CROSS_$1_3], [$CROSS_$1], [yes])
                if ! test "x$CROSS_$1_3" = "xyes"; then
                    AC_MSG_ERROR([no suitable CROSS_$1])
                fi
            fi
        fi
    fi
    AC_SUBST(CROSS_$1, $CROSS_$1)
])

GET_BINUTILS_PROG(ADDR2LINE, addr2line, addr2line)
GET_BINUTILS_PROG(AR, ar, archiver)
GET_BINUTILS_PROG(CXXFILT, c++filt, c++filt)
GET_BINUTILS_PROG(LD, ld, linker)
GET_BINUTILS_PROG(NM, nm, nm)
GET_BINUTILS_PROG(OBJCOPY, objcopy, objcopy)
GET_BINUTILS_PROG(OBJDUMP, objdump, objdump)
GET_BINUTILS_PROG(RANLIB, ranlib, ranlib)
GET_BINUTILS_PROG(READELF, readelf, readelf)
GET_BINUTILS_PROG(SIZE, size, size)
GET_BINUTILS_PROG(STRINGS, strings, strings)
GET_BINUTILS_PROG(STRIP, strip, strip)

AC_CONFIG_FILES([$ARCHITECTURE-toolchain.mk:toolchain.mk.in])
AC_OUTPUT
