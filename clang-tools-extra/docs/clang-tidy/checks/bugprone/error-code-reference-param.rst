.. title:: clang-tidy - bugprone-error-code-reference-param

bugprone-error-code-reference-param
===================================

A common error handling pattern, love it or hate it, is to pass a
std::error_code as a l-value reference to a function as the last
function parameter.

.. code-block:: c++

  void Init(std::error_code& ec) {
    ec.clear();

    InitSubsystem(ec);
    if (ec) {
      return;
    }

    if (!InValidState()) {
      ec = make_error_code(std::errc::state_not_recoverable);
    }
  }

Here an initialization function is given a `std::error_code&`, and the
expectation is that it is initialized if an error state has been
detected. The caller can then inspect the error_code object it passed
to Init() to see if an error has occurred.

Another pattern which is seen in some libraries like Boost.Asio is to
pass error_code to a callback function as a const reference. The
intent is to notify the callback that an error has occurred.

So there are two common, valid uses of std::error_code.

1. To bubble an error up the call stack by passing it as a l-value
   reference.
2. To notify a callback function that an error has occurred.

A common mistake is to try to use std::error_code to bubble an error
up the call stack, but accidentally pass it by value.

.. code-block:: c++

  void Init(std::error_code ec) {
    ec.clear();

    InitSubsystem(ec);
    if (ec) {
      return;
    }

    if (!InValidState()) {
      ec = make_error_code(std::errc::state_not_recoverable);
    }
  }

Now the caller of `Init` will never find out if some part of the
initialization process has failed.

This check tries to prevent this mistake in the use of std::error_code
by enforcing that std::error_code must be const if it is passed to a
function, unless it is being passed as a l-value reference.
