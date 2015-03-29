# Final Project Assignment 2: Exploration (FP2) 

### My Library: racket/gui, openssl

For this exploration, I used the racket/gui library, along with a collection of libraries needed for email access to make a simple email client.
I first created the necessary getters for accessing the email information, and then I constructed the gui for the client which will fetch and display the ten most recent emails in your inbox.

The code I wrote is as follows:
```
#lang racket/gui
(require racket/gui/base
         openssl
         net/head
         net/imap
         racket/match)

;;Constants
(define imap-server "imap.gmail.com")
(define imap-port-no 993)
(define username "--------")
(define password "--------")
(define mailbox-name "INBOX")


;;Set-up Mailbox
(define (acquire)
  (let-values ([(in out) (ssl-connect imap-server
                                       imap-port-no)])
    (imap-connect* in out username password mailbox-name)))

(define-values (imap count recent) (acquire))

;;Set-up complete

(define (get-message-count)
  (imap-reselect imap mailbox-name)
  (imap-messages imap))

(define (get-message n)
  (car (imap-get-messages imap (list n) '(uid header body flags))))

(define (get-uid message)
  (car message))
(define (get-header message)
  (cadr message))
(define (get-header-legible message)
  (extract-all-fields (get-header message)))
(define (get-body message)
  (caddr message))
(define (get-flags message)
  (cadddr message))

(define (get-date message)
  (cdr (findf (lambda (str)
                (regexp-match #rx"Date" (car str)))
              (get-header-legible message))))

(define (get-to message)
  (cdr (findf (lambda (str)
                (regexp-match #rx"To" (car str)))
              (get-header-legible message))))
(define (get-from message)
  (cdr (findf (lambda (str)
                (regexp-match #rx"From" (car str)))
              (get-header-legible message))))
(define (get-subject message)
  (cdr (findf (lambda (str)
                (regexp-match #rx"Subject" (car str)))
              (get-header-legible message))))
;;=======================================================================;;

(define frame (new frame% [label "Schemail"]))

(define panel (new horizontal-panel% [parent frame]
                   [spacing 50]))
(define messages-panel (new horizontal-panel% [parent frame]
                            [border 5]))
(define date-panel (new vertical-panel% [parent messages-panel]
                        [style '(border)]
                        [spacing 10]
                        [border 5]))
(define from-panel (new vertical-panel% [parent messages-panel]
                           [style '(border)]
                           [spacing 10]
                           [border 5]))
(define subject-panel (new vertical-panel% [parent messages-panel]
                        [style '(border)]
                        [spacing 10]
                        [border 5]))

(define msg1 (new message% [parent panel]
                  [label "0 emails!"]
                  [auto-resize #t]))
(new button% [parent panel]
     [label "Refresh"]
     [callback (lambda (button event)
                 (send msg1 set-label (string-append (number->string (get-message-count)) " emails!"))
                 (for ([child (send from-panel get-children)])
                   (send from-panel delete-child child))
                 (for ([child (send subject-panel get-children)])
                   (send subject-panel delete-child child))
                 (fetch-messages))])
(define (fetch-messages)
  (new message% [parent date-panel]
       [label "Date:"])
  (for ([i 10])
    (new message% [parent date-panel]
         [label (bytes->string/utf-8 (get-date (get-message (- (get-message-count) i))))]))
  (new message% [parent subject-panel]
       [label "Subject:"])
  (for ([i 10])
    (new message% [parent subject-panel]
         [label (bytes->string/utf-8 (get-subject (get-message (- (get-message-count) i))))]))
  (new message% [parent from-panel]
       [label "From:"])
  (for ([i 10])
    (new message% [parent from-panel]
         [label (bytes->string/utf-8 (get-from (get-message (- (get-message-count) i))))]))
  )

(send frame show #t)
```
* This is the gui before fetching the emails:
* ![Imgur](http://i.imgur.com/WXa5DrO.png)
* And this is after:
* ![Imgur](http://i.imgur.com/yOaP13T.png)
